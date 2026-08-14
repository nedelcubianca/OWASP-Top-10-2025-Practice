# A05:2025 Injection — Explained Simply

## What Is an "Injection" Vulnerability?

Your application often takes user input and passes it to some other system to do work — like a **database** (to fetch data), the **command line** (to run system commands), or a **browser** (to render a page). These systems are called **"interpreters"** because they read instructions and execute them.

An **injection flaw** happens when your application takes **untrusted user input** and hands it to one of these interpreters **without properly separating "this is just data" from "this is a command to run."** The interpreter ends up **treating part of the user's input as an actual command**, instead of just as harmless data — letting the attacker control what the interpreter does.

**Simple analogy:** imagine a form letter that says *"Dear [Name],"* and you let anyone type whatever they want into `[Name]`. If someone types `"John. Also, please transfer $1000 to me."`, and your system blindly inserts that whole text as if it were part of the intended message and then acts on it, you've been "injected." Injection is the software version of this — the "data" the user typed secretly becomes "instructions" your system follows.

## When Is an Application Vulnerable?

- User input is **not checked, filtered, or cleaned up** before being used
- The application builds queries/commands by **directly gluing user input into them** (instead of keeping data and commands properly separate)
- Even when using database tools like **ORMs** (a layer that's supposed to make database access safer), unfiltered input is still used in a way that leaks extra data
- Raw, potentially dangerous input gets **directly combined ("concatenated")** into a SQL query, system command, or stored procedure

**Common types of injection:** SQL (databases), NoSQL (non-relational databases), OS command (system commands), ORM, LDAP (directory services), and EL/OGNL (used in some Java frameworks). **They're all the same underlying concept**, just applied to different interpreters.

**How it's found:** by reviewing source code manually, plus automated testing (including "fuzzing" — throwing random/malformed input at the app to see what breaks) across all inputs — form fields, URLs, cookies, JSON, etc. Automated security testing tools built into the development pipeline help catch these before the app goes live.

**Related but separate topic:** injection attacks targeting AI/LLM systems (like "prompt injection") are covered in a different OWASP list (the LLM Top 10), not this one.

## How to Prevent It

**The best fix: keep data and commands completely separate**, so user input can never be misinterpreted as a command:

- Use a **safe API** — ideally one that avoids the raw interpreter entirely, or uses a **parameterized interface** (where data is passed in as a distinct "value" rather than glued into the command text), or use an **ORM** properly.
  - ⚠️ Caution: even "parameterized" stored procedures can still be vulnerable if the underlying database code itself builds a query by gluing text together internally.

**If you truly can't fully separate data from commands**, fallback (weaker) options:
- **Validate input on the server** (check that it matches an expected safe format) — though this isn't a complete fix, since some inputs legitimately need special characters (e.g., free-text fields)
- **Escape special characters** using the correct method for that specific interpreter (i.e., mark certain characters as "just data, not a command")
  - ⚠️ Note: things like table names or column names in SQL generally **can't be safely escaped** — letting users control these directly is dangerous (common issue in reporting tools)
- ⚠️ These fallback techniques are **error-prone** and can break if the underlying system changes slightly — they're a last resort, not a real solution.

## Example Attack Scenarios (Explained)

### Scenario #1: Classic SQL Injection
The vulnerable code:
```java
String query = "SELECT * FROM accounts WHERE custID='" + request.getParameter("id") + "'";
```
This builds a SQL query by **directly pasting the user's input** into the query text. Normally, `id` would be something like `12345`. But an attacker instead sends:
```
http://example.com/app/accountView?id=' OR '1'='1
```
This changes the actual query the database receives to:
```sql
SELECT * FROM accounts WHERE custID='' OR '1'='1'
```
Since `'1'='1'` is **always true**, the query's meaning is completely hijacked — instead of returning one customer's account, it **returns every single account in the table**. Worse attacks using this same technique could modify, delete data, or trigger other database functions.

### Scenario #2: "Safer" Framework, Same Mistake (HQL)
Even when using a framework meant to be safer than raw SQL (Hibernate Query Language, HQL), the same mistake — **gluing user input directly into the query string** — recreates the same vulnerability:
```java
Query HQLQuery = session.createQuery("FROM accounts WHERE custID='" + request.getParameter("id") + "'");
```
An attacker sends: `' OR custID IS NOT NULL OR custID='`, which again manipulates the query logic to **bypass the intended filter and return all accounts**. **Lesson:** using a "safer" framework doesn't automatically protect you — if you still concatenate raw user input into a query string, you're vulnerable regardless of the framework.

### Scenario #3: OS Command Injection
The vulnerable code takes user input and pastes it directly into a system command:
```java
String cmd = "nslookup " + request.getParameter("domain");
Runtime.getRuntime().exec(cmd);
```
This is meant to look up a domain name using the `nslookup` command. But an attacker sends as input:
```
example.com; cat /etc/passwd
```
On many systems, the `;` character means **"end this command, now start a new one."** So the actual command executed becomes:
```
nslookup example.com; cat /etc/passwd
```
The server runs `nslookup example.com` **and then also runs `cat /etc/passwd`** — a command that displays the contents of a sensitive system file. The attacker has successfully **injected and executed an arbitrary command** on the server, completely unrelated to the original intended function.

# What Is an ORM?

**ORM = Object-Relational Mapping.**

Normally, to talk to a database, you'd write raw SQL text like:
```sql
SELECT * FROM accounts WHERE custID = '12345'
```

An **ORM is a tool/library that lets you interact with the database using your programming language's normal objects and methods instead of writing raw SQL text yourself.** For example, instead of writing SQL, in a language like Python using an ORM, you might write something like:
```python
account = Account.objects.get(custID=user_input)
```

Behind the scenes, the ORM **translates this into SQL for you**. The key benefit: **you never manually type or glue together the SQL string yourself** — the ORM handles converting your input into a safe database query internally. Since you're not the one building raw SQL text, there's much less risk of accidentally creating an injection vulnerability... **as long as you use the ORM correctly** (mentioned as "use an ORM properly" — meaning you still shouldn't take user input and manually build a raw query string even *within* the ORM's tools, which some ORMs technically allow you to do if misused).

**Think of it like this:** instead of writing a letter by hand (raw SQL, where a mistake could let someone slip in extra "commands"), you fill out a structured form with clearly labeled fields (ORM), and a separate, trusted process (the ORM) converts your form into the final letter safely.

---

# "Keep Data and Commands Completely Separate" — Explained With a Clear Example

## The Core Problem, Restated Simply
When you build a database query by **directly pasting user input into the query text itself**, the database can't tell the difference between:
- "This part is a legitimate command/structure"
- "This part is just a piece of data the user typed"

It all just looks like **one long string of text** to the database. If the user's input happens to contain special characters (like a quote mark `'`), it can **break out of being "just data"** and start being interpreted as part of the actual command — which is exactly what happened in Scenario #1 (`' OR '1'='1`).

## The Bad Way (Concatenation) — What NOT to Do
```java
String query = "SELECT * FROM accounts WHERE custID='" + userInput + "'";
```
Here, `userInput` is **glued directly into the command text.** The database receives one single string, and has no way of knowing where "the command" ends and "the user's data" begins — it just reads the whole thing as one instruction.

## The Good Way (Parameterized Queries) — What TO Do
A **parameterized query** (also called a "prepared statement") works completely differently. You write the query with a **placeholder** instead of directly inserting the user's value:
```java
String query = "SELECT * FROM accounts WHERE custID = ?";
pstmt.setString(1, userInput);
```

**Here's the critical difference:** the query structure (`SELECT * FROM accounts WHERE custID = ?`) is sent to the database **first, separately, on its own** — the database compiles/understands this as "the command" *before* it ever sees the user's actual value. **Only afterward** is the user's input sent over, explicitly labeled as "this is a value to slot into that `?` spot — treat it purely as data, never as part of the command structure, no matter what characters it contains."

So even if the attacker sends `' OR '1'='1` as their input, the database doesn't reinterpret the command — it simply treats the entire string `' OR '1'='1` **literally**, as if someone's actual customer ID were that odd-looking piece of text. It will just search for an account with that literal (nonsensical) ID and find nothing — **the special characters have no power to change the command**, because the command was already locked in before the data ever arrived.

## Why This Is "The Best Fix"
By using a **safe API** (like parameterized queries) or a properly-used **ORM**, you architecturally **make it impossible** for user input to be misread as a command — not because you're carefully filtering out "dangerous" characters (which can be error-prone and easy to miss edge cases for), but because **the system itself never treats input as anything other than pure data**, by design.

## The Warning About Stored Procedures
A **stored procedure** is a saved, reusable block of database code that lives inside the database itself. People often assume "if I use a stored procedure, I'm automatically safe from injection." **This isn't always true.**

The warning says: even if *you* call a stored procedure in a parameterized way (correctly, from your application code), **the vulnerability can still exist if the stored procedure's own internal code is badly written** — for example, if the procedure itself takes an input and then **builds a new SQL query by gluing text together internally**, using commands like `EXECUTE IMMEDIATE` (Oracle) or `exec()` (SQL Server) to run that dynamically-built text.

**In other words:** you did everything right on your end (parameterized call), but the stored procedure itself reintroduces the exact same "gluing data into command text" mistake **internally**, recreating the vulnerability one layer deeper. This is why the text says the danger can exist "if the underlying database code itself builds a query by gluing text together" — the safety only holds if **every layer**, all the way down, avoids concatenating user input into command text.
