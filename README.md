# Postman API Tests: Restful Booker

[![Postman API Tests](https://github.com/da96vit/postman-api-tests/actions/workflows/newman.yml/badge.svg)](https://github.com/da96vit/postman-api-tests/actions/workflows/newman.yml)

An automated **Postman** test collection for the public [Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html) REST API. It runs end to end in the Postman Collection Runner or on the command line with **Newman**, and it runs in **GitHub Actions** on every push.

**14 requests · 39 assertions · runs in about 5 seconds**

(Newman's summary lists 16 requests: the 14 above plus 2 helper calls from scripts that create and delete the booking used by the negative cases.)

## What is covered

| Folder | Requests | Checks |
|---|---|---|
| **01 Health** | `GET /ping` | Status 201 |
| **02 Auth** | `POST /auth` valid and invalid credentials | Token issued and saved; "Bad credentials" without a token |
| **03 Booking CRUD** | Create → Get → Search → PUT → PATCH → Delete → Get deleted | JSON schema validation, data echo, partial vs full update, 404 after delete |
| **04 Negative cases** | No token, invalid token (on a booking created for the check), unknown id, missing required fields | 403, 403, 404, 500 and no booking id (see known defects) |
| *(every request)* | collection-level test | Response time under 5 s |

## Known API defects

Restful Booker is a practice API with deliberate quirks. The 500 and 201 statuses are asserted exactly (so a change in behaviour fails the run instead of passing silently); the bad-credentials case is checked through the response body:

| Request | Actual | Expected from a correct REST API |
|---|---|---|
| `POST /booking` with missing required fields | **500** Internal Server Error | 400 Bad Request |
| `POST /auth` with a wrong password | **200** with `{"reason": "Bad credentials"}` | 401 Unauthorized |
| `DELETE /booking/{id}` success | **201** Created | 200 OK or 204 No Content |

## Techniques used

- **Request chaining:** the token and `bookingId` are saved to collection variables and reused by later requests
- **Dynamic test data:** a pre-request script generates a unique guest name, so the search test can find the booking it created
- **Independent negative cases:** the 403 checks run against a booking created just for them (and deleted afterwards), not a fixed id that other users of the public API may have deleted
- **JSON Schema validation** of the create-booking response
- **Environment file** keeps the URL and credentials separate from the collection
- **CI/CD:** Newman in GitHub Actions with an HTML report (htmlextra) and JUnit XML, saved as build artifacts

## Run it

**In Postman:** import `collections/restful-booker.postman_collection.json` and `environments/restful-booker.postman_environment.json`, select the environment, then **Run collection**.

**From the command line:**

```bash
npm ci
npm test            # HTML report -> reports/report.html
```

## Project structure

```
├── collections/restful-booker.postman_collection.json
├── environments/restful-booker.postman_environment.json
├── .github/workflows/newman.yml
└── package.json
```

> The credentials in the environment file are the public demo credentials documented by Restful Booker.

## About me

**Davit Jikurashvili**, Senior QA Engineer · API, web, mobile (iOS) and database testing.
[Upwork](https://www.upwork.com/freelancers/~01027aece733fa0c7b) · [LinkedIn](https://www.linkedin.com/in/davit-jikurashvili-801a2319a/) · UI + API automation with Playwright: [playwright-qa-portfolio](https://github.com/da96vit/playwright-qa-portfolio)
