# Security & Secure Coding Practices

**Level:** 🔥 Advanced  
**Tags:** `security`, `OWASP`, `XSS`, `SQL-injection`, `authentication`, `encryption`, `best-practices`, `interview`

---

## Why this matters

Security vulnerabilities in production applications don't just cause embarrassing headlines—they destroy companies, expose millions of users' data, trigger regulatory fines, and end careers. Every senior engineer must understand security because a single SQL injection or XSS vulnerability can compromise an entire system, regardless of how elegant the architecture or how fast the code.

**Security bugs have asymmetric consequences.** I've seen a single forgotten input validation allow attackers to dump entire customer databases. One XSS vulnerability in an admin panel led to a full account takeover attack affecting 50,000 users. A hardcoded API key in source code (committed to GitHub) resulted in $40,000 of unauthorized cloud charges in 48 hours. These weren't sophisticated nation-state attacks—they were Script Kiddie-level exploits that succeeded because developers didn't follow basic secure coding practices. The consequences were catastrophic: regulatory investigations, class-action lawsuits, customer trust destroyed, executives fired.

**Interviews test security heavily for senior roles because it's non-negotiable.** When an interviewer asks "How would you prevent SQL injection?" or "What's the difference between authentication and authorization?", they're checking if you understand that security isn't optional or something "handled by the framework." They want to know if you'll write code that gets the company sued. I've seen stellar candidates rejected because they couldn't articulate basic security principles, even when their algorithm skills were excellent. At senior levels, the assumption is you won't introduce vulnerabilities that cost the company millions.

**The OWASP Top 10 represents the most common devastating vulnerabilities.** Injection attacks (SQL, NoSQL, LDAP), broken authentication, sensitive data exposure, XML external entities (XXE), broken access control, security misconfiguration, XSS, insecure deserialization, insufficient logging, and known vulnerable components. These aren't theoretical—they're the actual attack vectors responsible for most breaches. Understanding and preventing these is table stakes for professional development. I've reviewed codebases where 8 of the OWASP Top 10 were present. That's not code—that's a ticking time bomb.

**Input validation failures are the root cause of most vulnerabilities.** "Never trust user input" is the golden rule. Every SQL injection, XSS, command injection, and path traversal vulnerability stems from treating untrusted input as safe. I've seen production APIs that directly interpolated user input into SQL queries, web forms that rendered user content without encoding, and file upload handlers that didn't validate file types or names. Each was a critical vulnerability waiting to be exploited. Senior engineers must internalize that all external input—query strings, form data, headers, cookies, file uploads—is potentially malicious.

**Authentication and authorization are constantly confused, with disastrous results.** Authentication is "who are you?" Authorization is "what can you do?" I've reviewed systems where authentication was strong (good password policies, MFA) but authorization was broken—any authenticated user could access any other user's data by changing a URL parameter. These "Broken Object Level Authorization" (BOLA) vulnerabilities are now the #1 API security risk per OWASP. The number of production systems where changing `?userId=123` to `?userId=124` exposes another user's data is shocking. This isn't a complex attack—it's high school level—but it persists because developers don't validate authorization at every access point.

**Encryption and hashing are frequently misused or misunderstood.** I've seen production systems that: stored passwords in plaintext, used MD5 for password hashing (broken since 2004), encrypted sensitive data with hardcoded keys in source code, used ECB mode (the weakest encryption mode), stored encryption keys next to encrypted data, and used symmetric encryption where digital signatures were needed. Each is a critical vulnerability. Understanding when to use hashing (passwords, integrity), symmetric encryption (data at rest), asymmetric encryption (key exchange, digital signatures), and how to manage keys properly is essential. The difference between SHA256 for file integrity versus bcrypt for passwords isn't academic—misuse creates vulnerabilities.

**Certificate and HTTPS misunderstandings create man-in-the-middle vulnerabilities.** I've debugged production systems that: disabled certificate validation "because it was causing errors in dev," used self-signed certificates in production without proper trust chains, ignored certificate expiration warnings, and sent sensitive data over HTTP "for testing" (which stayed that way in production). These aren't hypothetical—every one is a real production vulnerability I've personally encountered. Understanding how TLS/SSL works, what certificates validate, and why you never disable certificate validation is critical for any system handling sensitive data.

**SQL injection is ancient (discovered 1998) yet still endemic.** It's #1 on the OWASP Injection list because it's devastating and still common. I've seen modern applications built in 2023 vulnerable to SQL injection because developers concatenated user input into queries instead of using parameterized queries. The fix is trivial—always use parameters—yet it's forgotten constantly. This vulnerability can expose entire databases, modify data, execute OS commands, and compromise entire servers. If you can't explain how to prevent SQL injection and demonstrate it with code, you're not qualified for senior roles.

**XSS (Cross-Site Scripting) is the vulnerability developers "know about" but still introduce.** I've reviewed React applications vulnerable to XSS because developers used `dangerouslySetInnerHTML` without sanitization, ASP.NET apps that rendered user input without encoding, and APIs that reflected user input in error messages without escaping. XSS allows attackers to execute JavaScript in users' browsers, steal session cookies, perform actions as the victim, and phish credentials. The fix is simple—always encode output—but requires discipline and understanding of different contexts (HTML, JavaScript, URL, CSS each require different encoding).

**Logging and monitoring are security controls, not just debugging tools.** When a breach occurs, the first question is "what happened?" Without comprehensive audit logs, you can't answer. I've investigated security incidents where we had no idea what data was accessed because the application didn't log security-relevant events. Conversely, I've also seen systems that logged sensitive data (passwords, credit cards, SSNs) in plaintext log files, creating a different vulnerability. Understanding what to log (authentication attempts, authorization failures, input validation failures), what not to log (secrets, PII), and how to protect logs is part of secure design.

**Dependency vulnerabilities are a supply chain attack vector.** Using outdated NuGet packages with known CVEs is like leaving the door unlocked. I've seen production systems using packages with critical vulnerabilities (remote code execution, authentication bypass) disclosed years earlier because nobody was monitoring dependencies. Tools like `dotnet list package --vulnerable` and GitHub Dependabot exist for a reason. Senior engineers ensure their dependency chain is monitored and updated, understanding that third-party code is just as dangerous as their own when vulnerable.

**Career and legal consequences are real.** Security breaches lead to: GDPR fines (up to 4% of global revenue), HIPAA violations (criminal penalties), PCI DSS failures (losing ability to process payments), SEC investigations (if publicly traded), and personal liability for executives and engineers. I know engineers whose careers were derailed by security incidents that could have been prevented with basic secure coding practices. Conversely, engineers who champion security, identify vulnerabilities in design reviews, and build secure-by-default systems are highly valued and promoted. Security expertise is a career differentiator at senior+ levels.

---

## Core Ideas

### 1. **The Golden Rule: Never Trust User Input**

**All external input is potentially malicious.**

"User input" includes:
- Query strings, POST data, JSON bodies
- HTTP headers (User-Agent, Referer, etc.)
- Cookies
- File uploads
- Database data (if from users originally)

**Defense strategy:**
1. **Validate** — whitelist acceptable values
2. **Sanitize** — remove/encode dangerous characters
3. **Parameterize** — use safe APIs (parameterized queries, ORMs)
4. **Encode** — escape output for context (HTML, JavaScript, SQL)

**Mental model:**  
"Assume every input is an attack. Validate on every boundary (client, API, database, rendering)."

---

### 2. **SQL Injection: Preventing Database Attacks**

**SQL injection** occurs when user input is concatenated into SQL queries, allowing attackers to execute arbitrary SQL.

```csharp
// ❌ VULNERABLE: SQL injection
string username = Request.Query["username"];
string query = $"SELECT * FROM Users WHERE Username = '{username}'";
var users = context.Database.SqlQuery<User>(query);

// Attack: username = "admin' OR '1'='1' --"
// Resulting query: SELECT * FROM Users WHERE Username = 'admin' OR '1'='1' --'
// Returns ALL users!
```

**Fix: Use parameterized queries**
```csharp
// ✅ SAFE: Parameterized query
string username = Request.Query["username"];
var users = context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Username = {0}", username)
    .ToList();

// Or better: use LINQ (ORM)
var users = context.Users
    .Where(u => u.Username == username)
    .ToList();
```

**Why this works:**
- Parameters are sent separately from query text
- Database treats them as data, not executable code
- Prevents injection regardless of input content

**Golden rule:**  
**Never concatenate user input into SQL. Always use parameters or ORMs.**

---

### 3. **XSS (Cross-Site Scripting): Preventing JavaScript Injection**

**XSS** occurs when user input is rendered in HTML without proper encoding, allowing attackers to inject malicious JavaScript.

```csharp
// ❌ VULNERABLE: XSS
string comment = GetUserComment();
<div>@Html.Raw(comment)</div>  // Renders unescaped HTML

// Attack: comment = "<script>alert(document.cookie)</script>"
// Browser executes the script, steals cookies!
```

**Fix: Always encode output**
```csharp
// ✅ SAFE: Razor automatically encodes by default
<div>@comment</div>  // Encoded: &lt;script&gt;...

// ✅ Explicit encoding
<div>@HttpUtility.HtmlEncode(comment)</div>
```

**XSS types:**
1. **Reflected XSS** — input immediately reflected (search results)
2. **Stored XSS** — input stored and displayed later (comments)
3. **DOM-based XSS** — client-side JavaScript manipulates DOM unsafely

**Defense:**
- **Encode output** — HTML encode for HTML context, JS encode for JavaScript context
- **Content Security Policy (CSP)** — restrict script sources
- **HttpOnly cookies** — prevent JavaScript access to session cookies

---

### 4. **Authentication: Verifying Identity**

**Authentication** answers: "Who are you?"

**Common mechanisms:**
- **Username/password** — traditional, requires secure storage
- **Multi-Factor Authentication (MFA)** — adds second factor (SMS, authenticator app)
- **OAuth/OpenID Connect** — delegate to identity provider (Google, Microsoft)
- **Certificate-based** — client certificates for API authentication

**Best practices:**
```csharp
// ✅ Password hashing (ASP.NET Core Identity)
using Microsoft.AspNetCore.Identity;

var passwordHasher = new PasswordHasher<User>();
string hashedPassword = passwordHasher.HashPassword(user, "password123");

// Verify password
var result = passwordHasher.VerifyHashedPassword(user, hashedPassword, "password123");
if (result == PasswordVerificationResult.Success)
{
    // Authenticated
}
```

**What NOT to do:**
- ❌ Store passwords in plaintext
- ❌ Use MD5 or SHA1 for passwords (use bcrypt, scrypt, Argon2, or PBKDF2)
- ❌ Roll your own crypto
- ❌ Use weak password policies

**JWT (JSON Web Tokens):**
```csharp
// Issue JWT token after successful authentication
var tokenHandler = new JwtSecurityTokenHandler();
var key = Encoding.ASCII.GetBytes(_config["Jwt:Key"]);

var tokenDescriptor = new SecurityTokenDescriptor
{
    Subject = new ClaimsIdentity(new[] { new Claim("id", user.Id.ToString()) }),
    Expires = DateTime.UtcNow.AddHours(2),
    SigningCredentials = new SigningCredentials(
        new SymmetricSecurityKey(key), 
        SecurityAlgorithms.HmacSha256Signature)
};

var token = tokenHandler.CreateToken(tokenDescriptor);
return tokenHandler.WriteToken(token);
```

---

### 5. **Authorization: Controlling Access**

**Authorization** answers: "What are you allowed to do?"

```csharp
// ✅ Role-based authorization
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser(int id)
{
    // Only admins can delete users
}

// ✅ Policy-based authorization
[Authorize(Policy = "CanEditProfile")]
public IActionResult EditProfile(int userId)
{
    // Check policy in Startup.cs
}

// ✅ Resource-based authorization
public async Task<IActionResult> EditPost(int postId)
{
    var post = await _context.Posts.FindAsync(postId);
    
    var authResult = await _authorizationService.AuthorizeAsync(
        User, post, "EditPostPolicy");
    
    if (!authResult.Succeeded)
        return Forbid();
    
    // User authorized to edit this specific post
}
```

**Common vulnerabilities:**
- **Broken Object Level Authorization (BOLA)** — failing to check if user owns resource
- **Insecure Direct Object References (IDOR)** — exposing internal IDs without authorization

```csharp
// ❌ VULNERABLE: No authorization check
public IActionResult GetOrder(int orderId)
{
    var order = _context.Orders.Find(orderId);
    return Ok(order);  // Any user can access any order!
}

// ✅ SECURE: Verify ownership
public IActionResult GetOrder(int orderId)
{
    var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
    var order = _context.Orders
        .FirstOrDefault(o => o.Id == orderId && o.UserId == userId);
    
    if (order == null)
        return NotFound();  // Or Forbid() to avoid info disclosure
    
    return Ok(order);
}
```

**Golden rule:**  
**Check authorization for EVERY resource access. Never trust client-side checks.**

---

### 6. **Encryption: Protecting Data**

**Three types of cryptography:**

**1. Hashing (One-Way, Integrity)**
- **Purpose:** Passwords, checksums, integrity verification
- **Algorithms:** bcrypt, Argon2, PBKDF2 (for passwords); SHA256, SHA512 (for integrity)
- **Cannot be decrypted**

```csharp
// Password hashing with PBKDF2
using (var deriveBytes = new Rfc2898DeriveBytes(password, salt, 100000, HashAlgorithmName.SHA256))
{
    byte[] hash = deriveBytes.GetBytes(32);
}
```

**2. Symmetric Encryption (Same Key for Encrypt/Decrypt)**
- **Purpose:** Data at rest, file encryption
- **Algorithms:** AES (standard), ChaCha20
- **Key management is critical**

```csharp
// AES encryption
using var aes = Aes.Create();
aes.Key = keyBytes;  // 128, 192, or 256 bits
aes.IV = ivBytes;    // Initialization vector

using var encryptor = aes.CreateEncryptor();
byte[] encrypted = encryptor.TransformFinalBlock(plaintext, 0, plaintext.Length);
```

**3. Asymmetric Encryption (Public/Private Key Pair)**
- **Purpose:** Key exchange, digital signatures, TLS
- **Algorithms:** RSA, ECC (Elliptic Curve)

```csharp
// RSA encryption
using var rsa = RSA.Create();
byte[] encrypted = rsa.Encrypt(data, RSAEncryptionPadding.OaepSHA256);
byte[] decrypted = rsa.Decrypt(encrypted, RSAEncryptionPadding.OaepSHA256);
```

**Best practices:**
- ❌ Don't roll your own crypto
- ✅ Use built-in .NET APIs or vetted libraries
- ✅ Store keys securely (Azure Key Vault, environment variables, NOT in code)
- ✅ Use authenticated encryption (AES-GCM) to prevent tampering

---

### 7. **HTTPS and Certificate Validation**

**HTTPS (TLS/SSL)** encrypts data in transit and verifies server identity.

**How it works:**
1. Client connects to server
2. Server presents certificate (public key + identity)
3. Client validates certificate (trusted CA, not expired, domain match)
4. Client and server negotiate encryption keys
5. All traffic is encrypted

**Certificate validation is critical:**
```csharp
// ❌ NEVER DO THIS IN PRODUCTION
ServicePointManager.ServerCertificateValidationCallback = 
    (sender, cert, chain, sslPolicyErrors) => true;  // Accepts ANY certificate!

// ✅ Use proper certificate validation
// Default behavior validates automatically
using var client = new HttpClient();
var response = await client.GetAsync("https://api.example.com");
```

**Best practices:**
- ✅ Always use HTTPS for sensitive data
- ✅ Enforce HTTPS with HSTS (HTTP Strict Transport Security)
- ✅ Use valid certificates (not self-signed in production)
- ✅ Monitor certificate expiration
- ❌ Never disable certificate validation

---

### 8. **Secure Configuration and Secrets Management**

**Never hardcode secrets in source code.**

```csharp
// ❌ VULNERABLE: Hardcoded secret
string apiKey = "sk_live_51H8XYZ...";  // Exposed if code is leaked

// ✅ SECURE: Environment variable
string apiKey = Environment.GetEnvironmentVariable("STRIPE_API_KEY");

// ✅ SECURE: Azure Key Vault (production)
var client = new SecretClient(new Uri(keyVaultUrl), new DefaultAzureCredential());
var secret = await client.GetSecretAsync("StripeApiKey");
string apiKey = secret.Value.Value;

// ✅ SECURE: User Secrets (development)
// dotnet user-secrets set "StripeApiKey" "sk_test_..."
string apiKey = _configuration["StripeApiKey"];
```

**Best practices:**
- ✅ Use environment variables or secrets management tools
- ✅ Different secrets for dev/staging/production
- ✅ Rotate secrets regularly
- ✅ Never commit secrets to source control (check with git-secrets, truffleHog)
- ✅ Use `.gitignore` for sensitive config files

---

## Examples

### Example 1 — SQL Injection prevention

```csharp
// ❌ VULNERABLE
string email = Request.Query["email"];
var users = context.Database.SqlQuery<User>(
    $"SELECT * FROM Users WHERE Email = '{email}'");

// ✅ SECURE: Parameterized query
var users = context.Database.SqlQuery<User>(
    "SELECT * FROM Users WHERE Email = @email",
    new SqlParameter("@email", email));

// ✅ BEST: Use ORM (LINQ)
var users = context.Users.Where(u => u.Email == email).ToList();
```

---

### Example 2 — XSS prevention in ASP.NET Core

```cshtml
@* ❌ VULNERABLE *@
<div>@Html.Raw(Model.UserComment)</div>

@* ✅ SECURE: Automatic encoding *@
<div>@Model.UserComment</div>

@* ✅ SECURE: Explicit encoding *@
<div>@Html.Encode(Model.UserComment)</div>
```

---

### Example 3 — Broken authorization (IDOR)

```csharp
// ❌ VULNERABLE: No ownership check
[HttpGet("{id}")]
public IActionResult GetDocument(int id)
{
    var doc = _db.Documents.Find(id);
    return Ok(doc);  // Any user can access any document by changing ID!
}

// ✅ SECURE: Verify ownership
[HttpGet("{id}")]
public IActionResult GetDocument(int id)
{
    var userId = int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier));
    var doc = _db.Documents.FirstOrDefault(d => d.Id == id && d.OwnerId == userId);
    
    if (doc == null)
        return NotFound();
    
    return Ok(doc);
}
```

---

### Example 4 — Secure password storage

```csharp
// ✅ Using ASP.NET Core Identity (recommended)
public async Task<IdentityResult> CreateUser(string email, string password)
{
    var user = new ApplicationUser { Email = email };
    var result = await _userManager.CreateAsync(user, password);
    return result;  // Password automatically hashed with PBKDF2
}

// ✅ Manual implementation (if not using Identity)
using System.Security.Cryptography;

public (byte[] hash, byte[] salt) HashPassword(string password)
{
    byte[] salt = RandomNumberGenerator.GetBytes(32);
    
    using var pbkdf2 = new Rfc2898DeriveBytes(
        password, salt, 100000, HashAlgorithmName.SHA256);
    
    byte[] hash = pbkdf2.GetBytes(32);
    
    return (hash, salt);
}
```

---

## Common Pitfalls

### ❌ 1. Trusting client-side validation

```javascript
// ❌ Client-side only (easily bypassed)
if (email.includes('@')) {
    submitForm();
}
```

**Always validate on the server:**
```csharp
// ✅ Server-side validation
if (!email.Contains('@'))
    return BadRequest("Invalid email");
```

---

### ❌ 2. Insufficient logging

```csharp
// ❌ No logging of security events
if (authFailed)
    return Unauthorized();

// ✅ Log security-relevant events
if (authFailed)
{
    _logger.LogWarning("Failed login attempt for {Email} from {IP}",
        email, HttpContext.Connection.RemoteIpAddress);
    return Unauthorized();
}
```

---

### ❌ 3. Exposing stack traces in production

```csharp
// ❌ Exposes internal implementation details
app.UseDeveloperExceptionPage();  // Don't use in production!

// ✅ Generic error page in production
if (app.Environment.IsProduction())
{
    app.UseExceptionHandler("/Error");
}
else
{
    app.UseDeveloperExceptionPage();
}
```

---

### ❌ 4. Using outdated dependencies

```bash
# ❌ Check for vulnerable packages
dotnet list package --vulnerable

# ✅ Update regularly
dotnet add package <PackageName> --version <NewVersion>
```

---

## Quick Self-Check

You should be able to answer these out loud:

1. What's SQL injection and how do you prevent it?
2. What's XSS and how do you prevent it?
3. What's the difference between authentication and authorization?
4. What's the difference between hashing and encryption?
5. Why should you never disable certificate validation?
6. What's IDOR/BOLA and how do you prevent it?
7. Where should you store API keys and secrets?
8. What password hashing algorithm should you use?

If you struggle with any of these, revisit the relevant section.

---

## Further Practice

1. Audit an existing codebase for SQL injection vulnerabilities.
2. Implement JWT authentication in an ASP.NET Core API.
3. Set up Azure Key Vault for secrets management.
4. Write unit tests that verify authorization checks.
5. Review the OWASP Top 10: https://owasp.org/Top10/

---

## Mini Interview Cheat Sheet

**Explain SQL injection prevention in 20 seconds:**
"Never concatenate user input into SQL queries. Always use parameterized queries or ORMs like Entity Framework that automatically parameterize inputs."

**Explain XSS prevention:**
"Always encode output when rendering user content. Use automatic encoding (Razor's `@` syntax) or explicit encoding functions. Set Content-Security-Policy headers."

**Authentication vs Authorization:**
- **Authentication** — Who are you? (login, JWT, OAuth)
- **Authorization** — What can you access? (roles, policies, resource ownership)

**OWASP Top 3:**
1. **Injection** (SQL, NoSQL, LDAP) — use parameterized queries
2. **Broken Authentication** — use strong passwords, MFA, secure sessions
3. **Sensitive Data Exposure** — encrypt at rest/transit, don't log secrets

**Secure secrets management:**
"Use environment variables for dev, Azure Key Vault/AWS Secrets Manager for production. Never hardcode. Never commit to source control. Use different secrets per environment."

---
