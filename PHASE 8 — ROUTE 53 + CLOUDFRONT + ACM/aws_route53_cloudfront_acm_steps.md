# PHASE 8 — ROUTE 53 + CLOUDFRONT + ACM

## Goal

By the end of this phase, you should understand how a real application moves from:

```text
User
  ↓
DNS
  ↓
TLS / HTTPS
  ↓
CloudFront / ALB
  ↓
S3 / EC2 / ECS
```

This phase connects three major AWS areas:

- **Route 53** — DNS and traffic routing
- **CloudFront** — global content delivery, caching, TLS termination, and origin protection
- **ACM (AWS Certificate Manager)** — TLS certificates for HTTPS

The important mental model is:

```text
Route 53
    |
    | "Where should this hostname go?"
    v
CloudFront / ALB
    |
    | "How should this request be served?"
    v
S3 / ALB / Backend
```

---

# 1. The Big Picture

Suppose your company owns:

```text
example.com
```

You want:

```text
https://www.example.com
```

for the frontend and:

```text
https://api.example.com
```

for the backend.

A common architecture is:

```text
                         Internet
                            |
                            v
                    +---------------+
                    |   Route 53    |
                    |      DNS      |
                    +---------------+
                       /          \
                      /            \
                     v              v
              www.example.com   api.example.com
                     |              |
                     v              v
                CloudFront         ALB
                     |              |
                     v              v
                     S3          EC2 / ECS
                                   |
                                   v
                                  RDS
```

Another architecture is:

```text
User
 |
 | HTTPS
 v
Route 53
 |
 v
CloudFront
 |
 +-------------------+
 |                   |
 v                   v
S3                  ALB
                     |
                     v
                  Backend
```

CloudFront can have multiple origins and cache behaviors, so one distribution can route different URL paths to different origins.

Example:

```text
example.com/
        |
        +-- /assets/*  -> S3
        |
        +-- /api/*     -> ALB
```

---

# 2. What Each Service Does

## Route 53

Route 53 answers DNS questions.

Conceptually:

```text
What IP/address/service belongs to:

api.example.com?
```

Route 53 can return an AWS resource target such as:

```text
CloudFront distribution
ALB
S3 website endpoint
API Gateway
```

using appropriate DNS records.

---

## CloudFront

CloudFront is a CDN.

Instead of every user going directly to your origin:

```text
User -> S3
```

you can have:

```text
User -> nearest CloudFront edge -> S3
```

CloudFront can:

- cache content
- terminate HTTPS
- use custom domains
- connect to private S3 origins using OAC
- forward selected headers/cookies/query strings
- route paths to different origins
- protect and accelerate origins
- restrict viewer protocols
- integrate with AWS WAF

---

## ACM

ACM manages TLS certificates.

For example:

```text
example.com
www.example.com
api.example.com
```

A certificate lets clients establish HTTPS connections such as:

```text
https://www.example.com
```

For AWS-integrated services such as CloudFront and Elastic Load Balancing, ACM public certificates can be used directly.

---

# 3. DNS Fundamentals

Before Route 53, understand DNS itself.

Suppose you type:

```text
https://api.example.com
```

Your browser needs to discover where:

```text
api.example.com
```

should go.

The domain name is hierarchical:

```text
api.example.com
│   │       │
│   │       └── Top-level domain
│   └────────── Domain
└────────────── Subdomain
```

For:

```text
www.example.com
```

- `www` = subdomain/hostname
- `example` = domain
- `.com` = top-level domain

DNS translates names into records.

---

# 4. Route 53 Hosted Zone

A hosted zone is a container for DNS records for a domain.

Example:

```text
example.com
```

Inside it:

```text
example.com
www.example.com
api.example.com
admin.example.com
```

You can have records such as:

```text
www.example.com -> CloudFront
api.example.com -> ALB
```

---

# 5. Public Hosted Zone

A public hosted zone is used for DNS names that should resolve through the public DNS system.

Example:

```text
example.com
```

Public records:

```text
www.example.com
api.example.com
```

are visible through public DNS resolution.

---

# 6. Private Hosted Zone

A private hosted zone is associated with one or more VPCs.

It is used for internal DNS.

Example:

```text
database.internal
api.internal
service.internal
```

Only resources in associated VPCs can resolve the private DNS namespace through the VPC DNS infrastructure.

Typical architecture:

```text
VPC
 |
 +-- EC2
 |
 +-- ECS
 |
 +-- RDS
 |
 +-- Private Hosted Zone
```

Example:

```text
db.internal.example.com
```

This can resolve to an internal resource without exposing that name through public DNS.

---

# 7. DNS Records

The important records for this phase are:

```text
A
AAAA
CNAME
Alias
```

---

# 8. A Record

An A record maps a hostname to an IPv4 address.

Example:

```text
api.example.com
        |
        v
203.0.113.10
```

Conceptually:

```text
A = IPv4 address
```

---

# 9. AAAA Record

AAAA maps a hostname to an IPv6 address.

Conceptually:

```text
AAAA = IPv6 address
```

Example:

```text
example.com
    |
    v
IPv6 address
```

If your application supports IPv6, AAAA records can be used.

---

# 10. CNAME Record

CNAME maps one DNS name to another DNS name.

Example:

```text
www.example.com
        |
        v
some-service.example.net
```

CNAME is useful when the destination itself is another hostname.

Important limitation:

A CNAME generally cannot be used at the DNS zone apex such as:

```text
example.com
```

That is one reason Route 53 Alias records are important.

---

# 11. Alias Record

An Alias record is a Route 53-specific feature for routing a name to supported AWS resources.

Examples:

```text
example.com -> CloudFront
www.example.com -> CloudFront
api.example.com -> ALB
```

Conceptually:

```text
Route 53 Alias
      |
      +----> CloudFront
      |
      +----> ALB
      |
      +----> other supported AWS targets
```

Alias records are especially useful for AWS architectures because they can point the zone apex at supported AWS resources.

---

# 12. A vs CNAME vs Alias

Think:

```text
A
|
+-- hostname -> IPv4

AAAA
|
+-- hostname -> IPv6

CNAME
|
+-- hostname -> another hostname

Alias
|
+-- Route 53 name -> supported AWS resource
```

For example:

```text
example.com
    |
    +-- Alias -> CloudFront

api.example.com
    |
    +-- Alias -> ALB
```

---

# 13. DNS TTL

TTL means:

```text
Time To Live
```

It controls how long DNS resolvers can cache a DNS answer.

Example:

```text
TTL = 300 seconds
```

means roughly:

```text
5 minutes
```

before a resolver should refresh the record.

Lower TTL:

```text
faster changes
more DNS queries
```

Higher TTL:

```text
fewer DNS lookups
slower DNS changes
```

TTL is not a magic guarantee that every client will instantly update at the exact second.

---

# 14. Route 53 Routing Policies

Route 53 supports different routing strategies.

Important examples:

```text
Simple
Weighted
Latency-based
Failover
Geolocation
Geoproximity
Multivalue answer
```

Do not memorize only the names.

Understand the problem each one solves.

---

# 15. Simple Routing

Simple routing is the basic case.

Example:

```text
www.example.com
        |
        v
CloudFront
```

One logical destination.

---

# 16. Weighted Routing

Weighted routing lets you distribute traffic among records.

Example:

```text
api.example.com

90% -> Version A
10% -> Version B
```

Conceptually:

```text
             +--> Backend A
User -> DNS -|
             +--> Backend B
```

This can be useful for controlled traffic distribution.

Do not confuse weighted DNS routing with an application-layer deployment mechanism. DNS caching and resolver behavior still matter.

---

# 17. Failover Routing

Failover routing is used for primary/secondary designs.

Example:

```text
Primary
   |
   v
Region A

Secondary
   |
   v
Region B
```

If the configured health-check logic identifies the primary as unhealthy, Route 53 can return the secondary record.

---

# 18. Latency-Based Routing

Latency-based routing chooses among supported destinations based on latency measurements.

Conceptually:

```text
User in Asia
     |
     v
Lower-latency destination

User in Europe
     |
     v
Lower-latency destination
```

This is useful when you have multiple regional deployments.

---

# 19. Route 53 Health Checks

A health check can monitor an endpoint.

Conceptually:

```text
Route 53
   |
   +---- health check ----> endpoint
```

Example:

```text
https://api.example.com/health
```

Possible response:

```text
200 OK
```

Route 53 can use health information with supported routing configurations.

Important:

A DNS health check is not the same thing as an ALB target-group health check.

ALB health checks determine whether targets are healthy for load balancing.

Route 53 health checks can influence DNS routing.

---

# 20. Route 53 + CloudFront

Typical frontend:

```text
User
 |
 v
Route 53
 |
 | Alias
 v
CloudFront
 |
 v
S3
```

The important distinction is:

```text
Route 53 = DNS
CloudFront = content delivery
S3 = origin/storage
```

Route 53 does not cache your JavaScript/CSS/images.

CloudFront does.

---

# 21. CloudFront Distribution

A CloudFront distribution is the main configuration object for a CDN deployment.

It defines things such as:

```text
Origins
Cache behaviors
Viewer protocol policy
TLS certificate
Custom domains
Cache policies
Origin request policies
OAC
Logging
WAF integration
```

Think:

```text
Distribution
|
+-- Origins
+-- Behaviors
+-- Policies
+-- TLS
+-- Security
```

---

# 22. CloudFront Origin

An origin is where CloudFront retrieves content when it needs it.

Examples:

```text
S3
ALB
EC2/custom HTTP server
API endpoint
```

Example:

```text
CloudFront
    |
    +---- S3 origin
```

or:

```text
CloudFront
    |
    +---- ALB origin
```

---

# 23. Origin vs Viewer

This distinction is extremely important.

Viewer:

```text
User -> CloudFront
```

Origin:

```text
CloudFront -> S3/ALB
```

Therefore:

```text
Viewer request
```

is not the same as:

```text
Origin request
```

Architecture:

```text
Browser
   |
   | viewer request
   v
CloudFront
   |
   | origin request
   v
Origin
```

---

# 24. CloudFront Cache

Suppose:

```text
GET /logo.png
```

User A requests it.

CloudFront doesn't have it cached:

```text
User
 |
 v
CloudFront
 |
 | cache miss
 v
S3
```

S3 returns the object.

CloudFront can cache it.

Then User B requests:

```text
GET /logo.png
```

CloudFront may serve it from cache:

```text
User B
  |
  v
CloudFront
  |
  | cache hit
  v
Response
```

The origin doesn't need to serve every request.

---

# 25. Cache Hit vs Cache Miss

Cache hit:

```text
Viewer
  |
  v
CloudFront
  |
  +-- object found
  |
  v
Response
```

Cache miss:

```text
Viewer
  |
  v
CloudFront
  |
  +-- object not found
  |
  v
Origin
  |
  v
CloudFront
  |
  v
Viewer
```

This is one of the most important CloudFront concepts.

---

# 26. Cache Policy

A cache policy controls the cache key and TTL-related caching behavior.

CloudFront can use values such as:

```text
URL path
Query strings
Headers
Cookies
```

as part of the cache key.

Example:

```text
/image.jpg?size=small
```

and:

```text
/image.jpg?size=large
```

could be treated as different cache keys if the relevant query string is included.

---

# 27. Why Cache Keys Matter

Suppose you accidentally include unnecessary values in the cache key:

```text
Cookie
User-Agent
random query parameter
```

Then many otherwise identical requests can become separate cache entries.

That can reduce cache efficiency.

Think:

```text
Good cache key
      |
      v
More reusable cached objects
```

versus:

```text
Overly variable cache key
      |
      v
More cache misses
```

---

# 28. Origin Request Policy

An origin request policy controls what CloudFront sends to the origin.

It can control:

```text
Headers
Cookies
Query strings
```

The key distinction:

```text
Cache Policy
    |
    +-- What participates in the cache key

Origin Request Policy
    |
    +-- What is forwarded to the origin
```

These policies are related but not identical.

---

# 29. Important Example

Suppose your backend needs:

```text
Authorization header
```

but you don't want every Authorization value to create a separate cached object.

You need to carefully design caching and origin forwarding.

For dynamic authenticated APIs, caching authenticated responses incorrectly can be dangerous.

A common strategy is:

```text
/api/*
    |
    v
CloudFront
    |
    +-- very limited/no caching
    |
    v
ALB
```

while static assets use aggressive caching:

```text
/assets/*
    |
    v
CloudFront
    |
    +-- long TTL
    |
    v
S3
```

---

# 30. CloudFront Cache Behaviors

Cache behaviors define how requests matching path patterns are handled.

Example:

```text
/*           -> S3
/api/*       -> ALB
/images/*    -> S3
```

Conceptually:

```text
example.com/
|
+-- /index.html
|       |
|       +--> S3
|
+-- /images/logo.png
|       |
|       +--> S3
|
+-- /api/users
        |
        +--> ALB
```

This is a powerful architecture.

---

# 31. Default Behavior

CloudFront has a default cache behavior.

Example:

```text
/*
```

If no more specific path behavior matches, the default behavior is used.

Specific behaviors can handle:

```text
/api/*
/static/*
/images/*
```

---

# 32. Viewer Protocol Policy

CloudFront can control whether users can access content using HTTP and/or HTTPS.

Common configuration:

```text
Redirect HTTP to HTTPS
```

or:

```text
HTTPS Only
```

For production applications, HTTPS should normally be the expected client-facing protocol.

---

# 33. TLS in CloudFront

When the user opens:

```text
https://www.example.com
```

the browser establishes TLS with the CloudFront viewer endpoint.

Conceptually:

```text
Browser
   |
   | TLS
   v
CloudFront
   |
   v
Origin
```

CloudFront needs an appropriate certificate for the custom hostname.

That certificate is commonly managed through ACM.

---

# 34. ACM — AWS Certificate Manager

ACM is AWS's certificate management service.

It can:

```text
Request certificates
Validate domain ownership
Manage certificates
Automatically renew eligible ACM certificates
Integrate with AWS services
```

For this phase, focus on:

```text
ACM
  |
  +-- Certificate
  +-- Domain validation
  +-- HTTPS
  +-- CloudFront
  +-- ALB
```

---

# 35. Public Certificate

Suppose you need:

```text
www.example.com
```

You request an ACM public certificate.

Possible names:

```text
example.com
www.example.com
api.example.com
```

You can also use wildcard names such as:

```text
*.example.com
```

subject to certificate/domain requirements.

---

# 36. Domain Validation

ACM needs proof that you control the domain.

DNS validation is generally the preferred approach when you can modify DNS.

Conceptually:

```text
ACM
 |
 | gives validation record
 v
Route 53
 |
 | DNS record proves control
 v
ACM validates domain
 |
 v
Certificate issued
```

---

# 37. DNS Validation

Example conceptual flow:

```text
Request certificate
       |
       v
ACM generates validation information
       |
       v
Create DNS validation record
       |
       v
Route 53
       |
       v
ACM verifies control
       |
       v
Certificate issued
```

With Route 53 integration, ACM can create the validation record for you when permissions and setup allow.

---

# 38. ACM Certificate and Region

This is a critical operational detail.

For a CloudFront distribution, the ACM certificate used by CloudFront must be in:

```text
us-east-1
```

For an Application Load Balancer, the ACM certificate is associated with the ALB in the same AWS Region as that ALB.

Therefore:

```text
CloudFront certificate
        |
        v
us-east-1
```

while:

```text
ALB certificate
        |
        v
ALB's Region
```

This is a common source of mistakes.

---

# 39. Custom Domain + CloudFront

Suppose:

```text
example.com
```

is your domain.

You want:

```text
https://www.example.com
```

Architecture:

```text
Browser
   |
   | https://www.example.com
   v
Route 53
   |
   | Alias
   v
CloudFront
   |
   | TLS certificate for www.example.com
   v
S3
```

---

# 40. Custom Domain + ALB

Backend:

```text
https://api.example.com
```

Architecture:

```text
Browser
   |
   v
Route 53
   |
   | Alias
   v
ALB
   |
   v
EC2 / ECS
```

The ALB HTTPS listener uses an ACM certificate for:

```text
api.example.com
```

---

# 41. CloudFront + Private S3

This is an important security architecture.

Do not make the S3 bucket public merely because CloudFront needs to read it.

Instead:

```text
User
 |
 v
CloudFront
 |
 | OAC
 v
Private S3
```

OAC means:

```text
Origin Access Control
```

CloudFront authenticates to the S3 origin.

The bucket remains private.

---

# 42. OAC Mental Model

Without OAC:

```text
CloudFront -> S3
```

You need a secure way for CloudFront to access the bucket.

With OAC:

```text
CloudFront
    |
    | authenticated origin request
    v
S3
```

The S3 bucket policy can authorize the CloudFront service principal for the specific distribution.

---

# 43. Recommended S3 Frontend Architecture

```text
                         Internet
                            |
                            v
                         Route 53
                            |
                            | Alias
                            v
                       CloudFront
                            |
                            | OAC
                            v
                    Private S3 Bucket
```

S3:

```text
Block Public Access = ON
```

CloudFront:

```text
OAC = enabled
```

This is the architecture you should practice.

---

# 44. Why Not Use S3 Website Hosting for This Architecture?

For a modern private S3 + CloudFront design, use the regular S3 bucket origin with OAC rather than treating the S3 website endpoint as the protected S3 origin.

Mental model:

```text
Private S3 bucket
      |
      v
CloudFront OAC
```

not:

```text
Public S3 website
      |
      v
CloudFront
```

---

# 45. Frontend Architecture

Build:

```text
User
 |
 v
Route 53
 |
 v
CloudFront
 |
 | OAC
 v
Private S3
```

Example domain:

```text
www.example.com
```

S3 contains:

```text
index.html
assets/
css/
js/
images/
```

---

# 46. Backend Architecture

A simple backend:

```text
User
 |
 v
Route 53
 |
 v
ALB
 |
 v
EC2 / ECS
 |
 v
RDS
```

Or with CloudFront in front:

```text
User
 |
 v
Route 53
 |
 v
CloudFront
 |
 v
ALB
 |
 v
EC2 / ECS
 |
 v
RDS
```

CloudFront is especially useful when you have a reason to place CDN/security behavior in front of the application.

Do not automatically add CloudFront to every dynamic API without considering caching, request forwarding, WebSockets, authentication, and operational requirements.

---

# 47. Full Company Architecture

A realistic frontend/backend design can be:

```text
                           Internet
                              |
                              v
                           Route 53
                              |
                 +------------+------------+
                 |                         |
                 v                         v
        www.example.com              api.example.com
                 |                         |
                 v                         v
             CloudFront                  ALB
                 |                         |
                OAC                        |
                 |                         v
                 v                    ECS / EC2
              Private S3                  |
                                           v
                                          RDS
```

TLS:

```text
www.example.com
      |
      v
CloudFront + ACM

api.example.com
      |
      v
ALB + ACM
```

---

# 48. Route 53 Does Not Provide HTTPS

This distinction is important.

Route 53:

```text
DNS
```

ACM:

```text
Certificate
```

CloudFront/ALB:

```text
TLS termination
```

So:

```text
Route 53 -> tells the client where to go
ACM -> provides the certificate
CloudFront/ALB -> uses the certificate for HTTPS
```

---

# 49. CloudFront Does Not Replace Route 53

CloudFront gives you a distribution hostname such as:

```text
d123example.cloudfront.net
```

You can use that directly, but for your company domain:

```text
www.example.com
```

Route 53 (or another DNS provider) points the custom hostname to CloudFront.

So:

```text
Route 53
    |
    v
CloudFront
```

are complementary services.

---

# 50. CloudFront Does Not Replace S3

S3:

```text
Storage / origin
```

CloudFront:

```text
Delivery / cache
```

Architecture:

```text
S3
 |
 | origin
 v
CloudFront
 |
 v
Users
```

---

# 51. CloudFront + ALB

CloudFront can also use an ALB as an origin.

Example:

```text
User
 |
 v
CloudFront
 |
 v
ALB
 |
 +--> EC2
 |
 +--> EC2
 |
 +--> ECS
```

This can combine:

```text
CDN
+
TLS
+
edge delivery
+
load balancing
```

---

# 52. Important TLS Layers

In some architectures there can be two TLS connections.

Example:

```text
Browser
   |
   | HTTPS
   v
CloudFront
   |
   | HTTPS
   v
ALB
   |
   v
Application
```

The first TLS connection protects:

```text
Viewer -> CloudFront
```

The second protects:

```text
CloudFront -> ALB
```

Do not assume that one TLS connection automatically means all network hops are encrypted.

---

# 53. CloudFront + ALB Hostname

Suppose:

```text
api.example.com
```

points to CloudFront.

CloudFront origin:

```text
my-alb-123.ap-south-1.elb.amazonaws.com
```

You must configure the CloudFront origin and HTTPS behavior correctly.

Also understand the difference between:

```text
Viewer Host
```

and:

```text
Origin Host
```

when designing application routing.

---

# 54. Cache Dynamic APIs Carefully

Static:

```text
/logo.png
/app.js
/styles.css
```

is usually straightforward to cache.

Dynamic:

```text
/api/me
/api/orders
/api/account
```

can contain user-specific data.

If caching is configured incorrectly, one user's response could potentially be served to another user.

Therefore:

```text
Static content
    |
    +-- strong caching is often useful

User-specific API
    |
    +-- carefully control/no caching
```

Never blindly enable caching for authenticated APIs.

---

# 55. Query Strings and Caching

Suppose:

```text
/products?id=10
/products?id=20
```

If `id` participates in the cache key:

```text
/product?id=10
```

and:

```text
/product?id=20
```

are different cached objects.

If it does not, CloudFront may treat them as the same cache key depending on the policy configuration.

Therefore understand:

```text
Cache Policy
        |
        v
Cache Key
        |
        v
Which requests share cached responses?
```

---

# 56. Cookies and Caching

Suppose:

```text
Cookie: session=user123
```

If user-specific cookies are incorrectly ignored by the cache design, responses can be mixed incorrectly.

For authenticated applications:

```text
Cookies
Authorization
Query strings
Headers
```

must be deliberately designed.

---

# 57. Headers

CloudFront requests can contain:

```text
Host
Authorization
User-Agent
CloudFront-specific headers
custom headers
```

Do not forward every header automatically without understanding the caching implications.

More forwarded values can increase origin complexity and potentially reduce cache reuse.

---

# 58. CloudFront OAC vs IAM

Remember:

```text
IAM
 |
 +-- controls AWS API/resource permissions
```

OAC:

```text
CloudFront
 |
 +-- controls authenticated access from CloudFront to S3
```

A private S3 bucket still uses an S3 bucket policy to authorize the CloudFront distribution.

---

# 59. Common Route 53 Mistakes

## Mistake 1 — Wrong hosted zone

You create:

```text
example.com
```

but modify records in another hosted zone.

Always verify:

```text
Hosted zone ID
Domain
Name servers
```

---

## Mistake 2 — Wrong record type

Do not randomly choose:

```text
A
CNAME
Alias
```

Understand the destination.

---

## Mistake 3 — Expecting DNS to update instantly

DNS is cached.

TTL and resolver caching affect propagation.

---

## Mistake 4 — Confusing Route 53 health checks with ALB health checks

They solve different problems.

---

# 60. Common CloudFront Mistakes

## Mistake 1 — Wrong origin

CloudFront:

```text
Origin -> wrong bucket/ALB
```

Result:

```text
403
404
502
```

---

## Mistake 2 — S3 bucket is private but OAC is missing

Result:

```text
403 AccessDenied
```

Check:

```text
OAC
Bucket policy
Distribution
Origin
```

---

## Mistake 3 — Certificate does not cover hostname

Example:

Certificate:

```text
example.com
```

Request:

```text
www.example.com
```

If the certificate doesn't cover the hostname, HTTPS will fail.

---

## Mistake 4 — Wrong ACM region

CloudFront:

```text
ACM us-east-1
```

ALB:

```text
ACM in ALB's region
```

---

## Mistake 5 — Cache not invalidated

You upload:

```text
app.js
```

but CloudFront continues serving a cached version.

Understand:

```text
TTL
Cache-Control
Invalidation
Versioned filenames
```

---

# 61. Cache Invalidation

Suppose:

```text
index.html
```

was cached.

You upload a new version.

You can invalidate paths such as:

```text
/*
```

or more targeted paths.

But frequent broad invalidations are not always the best design.

A common frontend deployment strategy is asset versioning:

```text
app.abc123.js
app.def456.js
```

Then new releases use new filenames.

This reduces reliance on invalidating long-lived immutable assets.

---

# 62. Recommended Frontend Caching Pattern

```text
index.html
    |
    +-- short/moderate caching

app.<hash>.js
    |
    +-- long caching

styles.<hash>.css
    |
    +-- long caching

image.<hash>.png
    |
    +-- long caching
```

Because the filename changes when the content changes.

---

# 63. HTTPS End-to-End Example

User enters:

```text
https://www.example.com
```

Flow:

```text
1. Browser resolves www.example.com
2. Route 53 returns CloudFront target
3. Browser connects to CloudFront
4. TLS handshake occurs
5. CloudFront checks cache
6. If cache hit -> return object
7. If cache miss -> CloudFront requests S3
8. S3 authorizes CloudFront through OAC
9. S3 returns object
10. CloudFront returns response
```

That is the complete mental model.

---

# 64. Backend HTTPS Example

User:

```text
https://api.example.com/users
```

Flow:

```text
1. DNS resolves api.example.com
2. Route 53 points to ALB or CloudFront
3. TLS is established
4. Request reaches ALB/CloudFront
5. Request is routed to backend
6. EC2/ECS handles request
7. Backend may query RDS
8. Response travels back
```

---

# 65. Project 1 — Route 53 + CloudFront + Private S3

Build:

```text
User
 |
 v
Route 53
 |
 v
CloudFront
 |
 | OAC
 v
Private S3
```

Requirements:

```text
S3 Block Public Access = ON
S3 Object Ownership = Bucket owner enforced
CloudFront OAC = enabled
HTTPS = enabled
Custom domain = enabled
ACM certificate = enabled
```

---

# 66. Project 1 — Step 1

Create or use a domain.

Example:

```text
example.com
```

If you own the domain through Route 53, create/manage the public hosted zone.

Understand:

```text
Hosted Zone
Name Servers
SOA
NS
```

Do not delete the required NS/SOA records accidentally.

---

# 67. Project 1 — Step 2

Create an S3 bucket.

Example:

```text
company-frontend-example
```

Keep:

```text
Block Public Access = ON
```

Do not enable public read access merely to make CloudFront work.

---

# 68. Project 1 — Step 3

Upload:

```text
index.html
app.js
styles.css
```

Example:

```html
<!doctype html>
<html>
<head>
  <title>AWS Phase 8</title>
</head>
<body>
  <h1>Route 53 + CloudFront + ACM</h1>
</body>
</html>
```

---

# 69. Project 1 — Step 4

Create CloudFront distribution.

Origin:

```text
S3 bucket
```

Important:

```text
Use S3 origin
Enable OAC
Keep bucket private
```

Do not use the S3 website endpoint for this private OAC design.

---

# 70. Project 1 — Step 5

Configure viewer protocol:

```text
Redirect HTTP to HTTPS
```

or:

```text
HTTPS only
```

For this lab, use:

```text
Redirect HTTP to HTTPS
```

so you can observe the redirect behavior.

---

# 71. Project 1 — Step 6

Configure the ACM certificate.

Request:

```text
www.example.com
```

or, if appropriate:

```text
example.com
www.example.com
```

Use:

```text
DNS validation
```

Then validate the domain.

For a CloudFront distribution, ensure the certificate is in:

```text
us-east-1
```

---

# 72. Project 1 — Step 7

Add the custom domain to CloudFront:

```text
www.example.com
```

Attach the ACM certificate.

CloudFront now knows:

```text
www.example.com
```

is an alternate domain name for the distribution.

---

# 73. Project 1 — Step 8

Create Route 53 record:

```text
www.example.com
```

Target:

```text
CloudFront distribution
```

Use:

```text
Alias
```

Test:

```text
https://www.example.com
```

---

# 74. Project 1 — Step 9

Test direct S3 access.

Try to access the object using a normal public S3 URL.

Expected:

```text
Access denied
```

That is correct for a private bucket.

Then test:

```text
https://www.example.com
```

Expected:

```text
Frontend loads
```

This proves:

```text
CloudFront
    |
    v
OAC
    |
    v
Private S3
```

is functioning.

---

# 75. Project 2 — Route 53 + ALB + EC2

Build:

```text
User
 |
 v
Route 53
 |
 v
ALB
 |
 v
EC2
```

Domain:

```text
api.example.com
```

---

# 76. Project 2 — ALB

Create:

```text
Application Load Balancer
```

Use at least the required subnets/AZs for the design.

Create:

```text
Target Group
```

Target:

```text
EC2
```

Health check:

```text
/
```

or:

```text
/health
```

---

# 77. Project 2 — Security Groups

ALB SG:

```text
Inbound:
443 from Internet
80 from Internet if redirecting HTTP
```

EC2 SG:

```text
Inbound:
Application port
Source = ALB Security Group
```

Do not use:

```text
0.0.0.0/0
```

for the application server's inbound port when the server should only receive traffic from the ALB.

This is the same security-group chaining principle you learned in Phase 6.

---

# 78. Project 2 — ACM

Request:

```text
api.example.com
```

Use DNS validation.

Attach the certificate to the ALB HTTPS listener.

Architecture:

```text
Browser
   |
 HTTPS
   v
Route 53
   |
 Alias
   v
ALB
   |
 HTTPS listener
   |
 ACM certificate
   |
   v
EC2
```

---

# 79. Project 3 — CloudFront + ALB

Build:

```text
User
 |
 v
Route 53
 |
 v
CloudFront
 |
 v
ALB
 |
 v
EC2 / ECS
```

This project is for learning:

```text
CloudFront origin = ALB
```

and:

```text
Viewer HTTPS
+
Origin HTTPS
```

where appropriate.

---

# 80. Project 4 — CloudFront Path Routing

Build one CloudFront distribution:

```text
CloudFront
|
+-- /*      -> S3
|
+-- /api/* -> ALB
```

Route 53:

```text
www.example.com
      |
      v
CloudFront
```

Architecture:

```text
                         CloudFront
                       /            \
                      /              \
                     v                v
                   S3                ALB
                                      |
                                      v
                                    ECS/EC2
                                      |
                                      v
                                     RDS
```

This is an excellent exercise because it combines:

```text
Route 53
CloudFront
OAC
S3
ALB
EC2/ECS
RDS
```

---

# 81. Project 4 — Important Cache Design

For:

```text
/*
```

you may use caching suitable for frontend assets.

For:

```text
/api/*
```

carefully configure:

```text
Allowed methods
Cache policy
Origin request policy
Headers
Cookies
Query strings
TTL
```

For user-specific APIs, start with a design that does not cache sensitive responses unless you have explicitly designed and tested the cache key and authorization behavior.

---

# 82. Troubleshooting — DNS

If:

```text
www.example.com
```

does not resolve:

Check:

```text
1. Correct hosted zone?
2. Correct record name?
3. Correct record type?
4. Correct Alias target?
5. Domain delegation correct?
6. Nameservers correct?
7. DNS caches/TTL?
```

---

# 83. Troubleshooting — ACM

If certificate remains:

```text
Pending validation
```

check:

```text
DNS validation record
Domain delegation
Hosted zone
Record name
Record value
```

If CloudFront does not let you select the certificate:

```text
Check ACM Region
```

For CloudFront:

```text
us-east-1
```

---

# 84. Troubleshooting — CloudFront 403

Potential causes:

```text
OAC configuration
S3 bucket policy
Wrong origin
Wrong object path
S3 permissions
CloudFront distribution configuration
```

Check the S3 bucket policy.

The policy should authorize the CloudFront distribution appropriately rather than making the entire bucket public.

---

# 85. Troubleshooting — CloudFront 404

Possible causes:

```text
Object doesn't exist
Wrong origin path
Wrong URL path
Wrong default root object
```

Example:

```text
/
```

may need:

```text
index.html
```

as the default root object for a static frontend.

---

# 86. Troubleshooting — ALB 502

If:

```text
CloudFront -> ALB
```

returns:

```text
502
```

investigate:

```text
ALB listener
Target group
Target health
Security groups
Backend port
Backend application
TLS configuration
```

Do not assume CloudFront is the root cause.

---

# 87. Troubleshooting — HTTPS Certificate Error

Check:

```text
Hostname
Certificate SANs
ACM status
Certificate region
CloudFront alternate domain name
ALB listener certificate
```

For example:

```text
Certificate:
example.com

Request:
api.example.com
```

The certificate may not cover the requested hostname.

---

# 88. Security Checklist

## Route 53

```text
[ ] Correct hosted zone
[ ] Correct records
[ ] Domain delegation verified
[ ] Health checks understood
[ ] Routing policies understood
```

## CloudFront

```text
[ ] HTTPS enabled
[ ] Correct origin
[ ] OAC enabled for private S3
[ ] Cache policy understood
[ ] Origin request policy understood
[ ] Sensitive API caching reviewed
[ ] Logging considered
[ ] WAF considered where appropriate
```

## S3

```text
[ ] Block Public Access ON
[ ] Bucket private
[ ] OAC used
[ ] Bucket policy restricted to CloudFront
[ ] Encryption enabled
```

## ACM

```text
[ ] Domain ownership validated
[ ] Certificate covers hostname
[ ] Correct Region
[ ] Renewal monitored
```

---

# 89. Operational Mental Model

When debugging:

```text
DNS problem?
    |
    v
Route 53

DNS works but HTTPS fails?
    |
    v
ACM / TLS / CloudFront or ALB

HTTPS works but content is 403?
    |
    v
CloudFront / OAC / S3 policy

Content is stale?
    |
    v
CloudFront cache / TTL / invalidation

API fails?
    |
    v
CloudFront behavior / ALB / target group / EC2/ECS

Backend fails?
    |
    v
EC2/ECS / Security Groups / RDS
```

---

# 90. The Most Important Distinctions

Memorize these:

```text
Route 53
    = DNS

CloudFront
    = CDN + edge delivery + viewer TLS + origin integration

ACM
    = certificates

S3
    = object storage

ALB
    = Layer 7 load balancing

OAC
    = authenticated CloudFront -> S3 origin access
```

---

# 91. Complete Request Flow — Frontend

```text
Browser
   |
   | https://www.example.com
   v
DNS Resolver
   |
   v
Route 53
   |
   | Alias
   v
CloudFront
   |
   | cache hit?
   |
   +------ YES ------> Browser
   |
   NO
   |
   v
S3
   |
   | OAC authorization
   v
Private S3
   |
   v
CloudFront
   |
   v
Browser
```

---

# 92. Complete Request Flow — Backend

```text
Browser
   |
   | https://api.example.com/users
   v
DNS
   |
   v
Route 53
   |
   v
CloudFront / ALB
   |
   v
Target Group
   |
   v
EC2 / ECS
   |
   v
RDS
```

---

# 93. What You Should Be Able to Explain Without Looking at Notes

You should be able to answer:

1. What is DNS?
2. What is a hosted zone?
3. What is an A record?
4. What is an AAAA record?
5. What is a CNAME?
6. What is a Route 53 Alias?
7. Why is Alias useful with CloudFront and ALB?
8. What is a CloudFront distribution?
9. What is an origin?
10. What is a cache behavior?
11. What is a cache policy?
12. What is an origin request policy?
13. What is a cache key?
14. What is a cache hit?
15. What is a cache miss?
16. What is OAC?
17. Why keep an S3 bucket private behind CloudFront?
18. What is ACM?
19. How does DNS validation work?
20. Why does CloudFront use an ACM certificate in us-east-1?
21. Where is an ACM certificate for an ALB created?
22. How does HTTPS work in this architecture?
23. Why is Route 53 different from ACM?
24. Why is CloudFront different from S3?
25. How would you troubleshoot a CloudFront 403?
26. How would you troubleshoot a certificate validation problem?
27. How would you design `/api/*` differently from `/assets/*`?

---

# 94. Phase 8 Labs

Complete these in order:

```text
LAB 1
Create a Route 53 public hosted zone
```

```text
LAB 2
Create A/AAAA/CNAME records and understand each
```

```text
LAB 3
Create an Alias to an AWS resource
```

```text
LAB 4
Create an ACM public certificate
```

```text
LAB 5
Validate the certificate using DNS validation
```

```text
LAB 6
Create a private S3 frontend bucket
```

```text
LAB 7
Create CloudFront distribution
```

```text
LAB 8
Configure OAC
```

```text
LAB 9
Configure ACM certificate for CloudFront
```

```text
LAB 10
Point Route 53 at CloudFront
```

```text
LAB 11
Test HTTP -> HTTPS
```

```text
LAB 12
Prove direct S3 access is blocked
```

```text
LAB 13
Prove CloudFront can access S3
```

```text
LAB 14
Deploy ALB + EC2 backend
```

```text
LAB 15
Attach ACM certificate to ALB
```

```text
LAB 16
Create api.example.com -> ALB
```

```text
LAB 17
Put CloudFront in front of ALB
```

```text
LAB 18
Create /api/* -> ALB and /* -> S3
```

```text
LAB 19
Test cache hits/misses
```

```text
LAB 20
Change S3 content and study caching/invalidation
```

---

# 95. Final Architecture

After completing this phase, your mental model should be:

```text
                                  Internet
                                      |
                                      v
                                   Route 53
                                      |
                    +-----------------+-----------------+
                    |                                   |
                    v                                   v
             www.example.com                      api.example.com
                    |                                   |
                    v                                   v
                CloudFront                              ALB
                    |                                   |
                   OAC                                  |
                    |                                   v
                    v                              EC2 / ECS
             Private S3                                  |
                                                        v
                                                       RDS
```

TLS:

```text
www.example.com
      |
      v
CloudFront
      |
      +-- ACM certificate in us-east-1

api.example.com
      |
      v
ALB
      |
      +-- ACM certificate in ALB Region
```

---

# 96. Final Mental Model

Think about a request in layers:

```text
1. DNS
   "Where should I go?"

2. TLS
   "Can I establish a trusted HTTPS connection?"

3. Edge/CDN
   "Can CloudFront serve this from cache?"

4. Origin access
   "Can CloudFront securely access the origin?"

5. Load balancing
   "Which backend target should receive this?"

6. Application
   "Can the application process this request?"

7. Database
   "Does the application need RDS?"
```

That gives you:

```text
Route 53
   ↓
ACM / TLS
   ↓
CloudFront
   ↓
OAC / Origin
   ↓
ALB
   ↓
EC2 / ECS
   ↓
RDS
```

This is the complete connection between the AWS topics you have learned so far.

---

# 97. Official AWS Documentation

Use the official documentation as your reference while doing the labs.

## Route 53

- Route 53 documentation:
  https://docs.aws.amazon.com/route53/

- Route 53 DNS routing:
  https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html

- Route 53 health checks:
  https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html

## CloudFront

- CloudFront documentation:
  https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/

- Cache policies:
  https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cache-key-understand-cache-policy.html

- Origin request policies:
  https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/origin-request-create-origin-request-policy.html

- Cache behaviors:
  https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/DownloadDistValuesCacheBehavior.html

## ACM

- ACM documentation:
  https://docs.aws.amazon.com/acm/

- Request public certificates:
  https://docs.aws.amazon.com/acm/latest/userguide/acm-public-certificates.html

- DNS validation:
  https://docs.aws.amazon.com/acm/latest/userguide/dns-validation.html

---

# Phase 8 Completion Checklist

```text
DNS
[ ] I understand DNS
[ ] I understand hosted zones
[ ] I understand public vs private hosted zones
[ ] I understand A
[ ] I understand AAAA
[ ] I understand CNAME
[ ] I understand Alias
[ ] I understand TTL
[ ] I understand health checks
[ ] I understand routing policies

CloudFront
[ ] I understand distributions
[ ] I understand origins
[ ] I understand viewer requests
[ ] I understand origin requests
[ ] I understand cache hits
[ ] I understand cache misses
[ ] I understand cache keys
[ ] I understand cache policies
[ ] I understand origin request policies
[ ] I understand cache behaviors
[ ] I understand OAC
[ ] I understand HTTPS

ACM
[ ] I understand TLS certificates
[ ] I understand DNS validation
[ ] I understand custom domains
[ ] I understand CloudFront certificate region
[ ] I understand ALB certificate region
[ ] I understand certificate renewal

Architecture
[ ] Route 53 -> CloudFront -> S3
[ ] Private S3 + OAC
[ ] Route 53 -> ALB -> EC2/ECS
[ ] CloudFront -> ALB
[ ] CloudFront path-based origin routing
[ ] Frontend HTTPS
[ ] Backend HTTPS
[ ] Cache troubleshooting
[ ] DNS troubleshooting
[ ] TLS troubleshooting
[ ] CloudFront 403 troubleshooting
```

# End of Phase 8
