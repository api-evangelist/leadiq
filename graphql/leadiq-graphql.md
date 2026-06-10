# LeadIQ GraphQL API

Public GraphQL Data API exposing LeadIQ's verified B2B contact and company intelligence. Supports people search by name + company / domain / LinkedIn URL / email, company search by name or domain, grouped and flat advanced search with ContactFilter and CompanyFilter inputs (industries, titles, seniorities, technologies, geography), prospect list management (createList, addProspectToList), account / usage / plan inspection, the workatoToken query for low-code integration, and submitPersonFeedback / markAsInvalid mutations for closed-loop data correction. Premium data fields (mobile phone) and ProfileFilter inputs (HasWorkEmail, HasVerifiedWorkEmail, HasWorkPhone, HasVerifiedWorkPhone, HasPersonalEmail) gate higher-cost lookups. Authentication is HTTP Basic with the API key as the username and an empty password.

**Endpoint:** https://api.leadiq.com/graphql

**Documentation:** https://developer.leadiq.com/

**GraphQL Properties:**

- GraphQL: https://api.leadiq.com/graphql
