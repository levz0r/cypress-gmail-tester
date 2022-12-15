<p align="center"><img alt="cypress-gmail-tester" src="https://user-images.githubusercontent.com/4564386/206307856-d2968539-54f8-4aa6-ad0f-da87881a7373.png" width="80%"/></p>

# cypress-gmail-tester

Use Cypress E2E tests to check Gmail-based inboxes using the [gmail-tester](https://github.com/levz0r/gmail-tester) library.

# Usage

1.  Install using `npm`:

```
npm install --save gmail-tester
```

2. Modify `cypress.config.json`

Create a task and call it `gmail:get-messages` in `cypress.config.json`, as shown here:

```js
const { defineConfig } = require("cypress");
const gmailTester = require("gmail-tester");
const path = require("path");

module.exports = defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      on("task", {
        "gmail:get-messages": async (args) => {
          const messages = await gmailTester.get_messages(
            path.resolve(__dirname, "credentials.json"),
            path.resolve(__dirname, "token.json"),
            args.options
          );
          return messages;
        }
      });
    },
  },
});
```
Make sure credentials.json and token.json file are located in cypress root folder (in the same location as `cypress.config.json`) .More information on credentials.json and token.json can be found <a href="https://github.com/levz0r/gmail-tester#how-to-get-credentialsjson">here</a>.

3. Call the task within any spec:

```js
/// <reference types="Cypress" />

describe("Email assertion:", () => {
  it("Using gmail_tester.get_messages(), look for an email with specific subject and link in email body", function () {
    // debugger; //Uncomment for debugger to work...
    cy.task("gmail:get-messages", {
      options: {
        from: "AccountSupport@ubi.com",
        subject: "Ubisoft Password Change Request",
        include_body: true,
        before: new Date(2019, 8, 24, 12, 31, 13), // Before September 24rd, 2019 12:31:13
        after: new Date(2019, 7, 23), // After August 23, 2019
      },
    }).then((emails) => {
      assert.isAtLeast(
        emails.length,
        1,
        "Expected to find at least one email, but none were found!"
      );
      const body = emails[0].body.html;
      assert.isTrue(
        body.indexOf(
          "https://account-uplay.ubi.com/en-GB/action/change-password?genomeid="
        ) >= 0,
        "Found reset link!"
      );
    });
  });
});
```
