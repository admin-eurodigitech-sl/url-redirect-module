# URL Rewrite Module for Multi-Tenant E-Commerce

## Overview

The **URL Rewrite Module** is designed to handle dynamic URL redirection for e-commerce platforms, focusing on Salesforce Commerce Cloud (SFCC). It allows for flexible, rule-based URL rewriting and redirection that can be configured via a Business Manager interface, supporting both simple and advanced use cases (e.g., site restructuring, product lifecycle management, A/B testing). The module will handle redirections using regular expressions (regex) and ensure the configuration is both scalable and highly performant, supporting features like in-memory caching and logging for analytics.

---

## Key Features & Requirements

1. **Admin Interface in Business Manager**:
   - **URL Configuration**:
     - Original URL path (regex supported).
     - Target URL path.
     - Redirect type (301 or 302).
     - Active/inactive status.
     - Optional start/end date range.
   - **Bulk Import/Export**:
     - CSV support for importing/exporting rules.
   - **Localization Support**:
     - Ability to define redirects based on site locale/language.

2. **Backend Storage**:
   - Use **MongoDB** for storing redirect rules:
     - Each rule will have fields for:
       - `regexPattern` (string): The regular expression to match the original URL.
       - `targetUrl` (string): The URL to redirect to.
       - `redirectType` (string): 301 or 302.
       - `active` (boolean): Whether the redirect is currently active.
       - `startDate` (Date, optional): When the redirect starts being active.
       - `endDate` (Date, optional): When the redirect stops being active.
       - `locale` (string, optional): The language/locale for localized redirects.
       - `priority` (integer, optional): The priority for sorting the redirects.

3. **Request Handling & URL Matching**:
   - **CDN Integration**:
     - Redirects should be processed at the CDN level for faster response times, avoiding hitting the origin server when possible.
   - For more complex rules (e.g., URL typos or dynamic content redirections), an **application-level redirect** should be implemented.
   - **Regex Matching**:
     - Native **Node.js `RegExp`** object can be used for matching patterns.
     - If more advanced and secure matching is required (e.g., protection against ReDoS attacks), **RE2** (from npm) can be utilized, as it offers faster and more secure regex processing.

4. **Performance Optimization**:
   - **In-Memory Caching**:
     - Use **Redis** or an equivalent key-value store for caching redirects.
     - Caching will reduce load by eliminating the need to check the MongoDB collection on every request.
     - Store frequently accessed redirects or the result of regex matches.
   - **Cache Misses**: For more complex or infrequently accessed redirects, fall back to MongoDB.

5. **Logging & Analytics**:
   - **Standardized Logs**:
     - Log every redirect, including:
       - **Rule ID** (identifier for the redirect rule).
       - **Timestamp**.
       - **Matched URL**.
       - **Target URL**.
     - **CDN Matched Rules**: If a rule is matched by the CDN, no logging should be performed by the application to avoid redundant logging.
   - Logs can be used for analytics (e.g., redirect frequency, user behavior, performance monitoring).

6. **Security Considerations**:
   - Ensure that the system is resistant to **ReDoS** (Regular Expression Denial of Service) attacks.
   - Provide input validation for user-defined regex patterns to prevent excessive computational complexity.

---

## Detailed Architecture

1. **Admin Interface**:
   - Implemented as a **Next.js** application integrated into the SFCC Business Manager.
   - Allows merchants to manage redirects via a clean, user-friendly interface.
   - Supports CRUD operations, bulk CSV import/export, and validation of regex patterns.

2. **Redirect Rule CRUD APIs**:
   - Expose RESTful APIs for managing redirects:
     - `GET /redirects`: Fetch all redirects.
     - `POST /redirects`: Create a new redirect.
     - `PUT /redirects/{id}`: Update an existing redirect.
     - `DELETE /redirects/{id}`: Remove a redirect.
     - **Bulk Import**: API to upload CSV files with redirect rules.
     - **Bulk Export**: API to download all redirect rules in CSV format.

3. **Backend Storage (MongoDB)**:
   - A **MongoDB collection** stores each redirect rule.
     - Fields:
       - `regexPattern` (string): The regular expression to match the original URL.
       - `targetUrl` (string): The URL to redirect to.
       - `redirectType` (string): 301 or 302.
       - `active` (boolean): Whether the redirect is currently active.
       - `startDate` (Date, optional): When the redirect starts being active.
       - `endDate` (Date, optional): When the redirect stops being active.
       - `locale` (string, optional): The language/locale for localized redirects.
       - `priority` (integer, optional): The priority for sorting the redirects.
   - Query MongoDB using regex to find a matching rule for the incoming request.

4. **Request Handling and CDN Integration**:
   - **Early Intercept**: When an incoming request is received, the system will first check the cache (Redis). If not found, it will query MongoDB.
   - **CDN-Level Caching**: If the CDN supports the redirect rule, handle it directly at the edge to minimize load on the application server.
   - **Application-Level Redirects**: For more complex or non-cacheable redirects, handle them at the application level using Node.js or SFCC’s middleware.

5. **Logging and Analytics**:
   - Use **structured logging** to capture detailed information about redirects:
     - **Rule ID**: Identifier for the redirect rule.
     - **Matched URL**: The URL being redirected.
     - **Target URL**: The destination URL.
     - **Timestamp**: When the redirect occurred.
   - Store logs in a central logging system (e.g., Elasticsearch, CloudWatch) for real-time analytics and monitoring.
   - **CDN Logging**: If the CDN handles a redirect, logs are not generated by the application.

---

## Use Case Scenarios

1. **Site Restructuring**:
   - When changing the URL structure of the site (e.g., moving products into new categories), use regex rules to map old URLs to new ones.
   - Example: `/old-category/product-name` → `/new-category/product-name`

2. **Product Lifecycle Management**:
   - When a product is discontinued or replaced, configure a redirect to the new product or category page.
   - Example: `/old-product` → `/new-product`

3. **Marketing Campaigns**:
   - Create simple, campaign-specific URLs that redirect to specific landing pages.
   - Example: `/campaign/holiday-sale` → `/landing-pages/holiday-sale`

4. **Brand Consolidation**:
   - Redirect users coming from old brand-specific domains to the new consolidated domain.
   - Example: `oldbrand.com/product` → `newbrand.com/product`

5. **URL Normalization**:
   - Ensure consistency by redirecting uppercase URLs to lowercase.
   - Example: `/Product/Item123` → `/product/item123`

6. **Geolocation-Based Redirects**:
   - Redirect users to country-specific versions of the site based on geolocation.
   - Example: `/product` → `/us/product` (for US users).

7. **A/B Testing**:
   - Dynamically redirect a portion of traffic to a variant page for A/B testing.
   - Example: `/product-page` → `/product-page-a` for a subset of users.

8. **SSL Enforcement**:
   - Automatically redirect all HTTP traffic to HTTPS.
   - Example: `http://example.com` → `https://example.com`

---

## Tech Stack

- **Backend**: Node.js, Express.js
- **Frontend**: Next.js for Business Manager UI
- **Storage**: MongoDB (for storing redirect rules)
- **Cache**: Redis (for in-memory caching)
- **Regex Matching**: Node.js `RegExp` or RE2
- **Logging**: Structured logging with integration to an analytics system (e.g., Elasticsearch)
- **Deployment**: Hosted on Salesforce Commerce Cloud or a suitable cloud infrastructure with CDN support (e.g., Cloudflare, Akamai).

---

## Conclusion

The **URL Rewrite Module** will provide e-commerce businesses with a flexible and performant solution for managing URL redirects. With features such as multi-tenant support, regex-based matching, localization, logging, and caching, this solution can meet a wide range of needs from SEO maintenance to marketing and A/B testing. By integrating with CDNs and utilizing in-memory caching, the module will also ensure high performance even for large e-commerce sites with complex redirect requirements.
