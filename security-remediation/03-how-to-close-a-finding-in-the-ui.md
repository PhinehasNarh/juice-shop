# How to close a security finding from the GitHub web interface

This page shows how to close a code scanning finding yourself, using only the GitHub website, with no command line. It is written so you can follow it click by click.

There are two ways a finding closes. It helps to know the difference before you start.

- Fixed. You change the code or upgrade the dependency, push the change, and the scan runs again. If the weakness is really gone, GitHub moves the alert to Closed with the reason Fixed on its own. You do not click anything.
- Dismissed. You decide the finding does not need a code change right now, for example because it is intentional, a false positive, or only used in tests. You tell GitHub to close it and you give a reason. The alert moves to Closed with the reason you chose. This is the manual path that the steps below describe.

## Who can do this

You need write access or the security alert permission on the repository. As the owner of this fork, you have it. If a teammate cannot see the Dismiss button, their role is the reason.

## Step by step: dismiss one finding

1. Open the repository on github.com.
2. Click the **Security** tab at the top of the repository.
3. In the left sidebar, click **Code scanning**. You now see the list of alerts. You can filter by tool, by rule, or by severity using the search bar, for example type `tool:CodeQL` or `severity:high`.
4. Click the title of the alert you want to close. This opens the alert detail page. The detail page shows the rule, the file and line, and the data flow that the tool found.
5. On the alert detail page, find the **Dismiss alert** button near the top right.
6. Click it. A small menu appears asking for a reason. Pick the reason that fits:
   - **Won't fix**. The finding is real but you accept the risk and will not change it. This is the right choice for the intentional Juice Shop vulnerabilities.
   - **Used in tests**. The flagged code only runs in tests, for example a fake token in a test file.
   - **False positive**. The tool is wrong and there is no real weakness here.
7. A comment box appears. Type a short reason, for example `Intentional training vulnerability, see security-remediation folder`. The comment is saved with the alert so anyone who looks later understands the decision.
8. Confirm. The alert now shows as Closed with your reason and your name.

That is the whole flow. It is the same for CodeQL, Semgrep, and Trivy alerts, because they all become code scanning alerts once they reach the Security tab.

## How to reopen a finding

If you change your mind, open the closed alert and click **Reopen alert**. It goes back to the open list. This is why dismissing is safe to practice. Nothing is lost.

## How to close many at once

On the Code scanning list page you can tick the checkbox next to several alerts, then use the **Dismiss** menu that appears at the top of the list to close them all with one reason. This is faster when you have a batch that share the same disposition.

## Two findings were left open for you to practice on

Most of the alerts in this repository were already dismissed with a documented reason. Two were left open on purpose so you can walk through the steps above yourself.

| Alert | Type | File and line | Suggested reason |
|------:|------|---------------|------------------|
| [#5](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/5) | Open redirect (CodeQL) | routes/redirect.ts:19 | Won't fix, intentional challenge |
| [#22](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/22) | Stack trace exposure (CodeQL) | routes/web3Wallet.ts:36 | Won't fix, intentional challenge |

To close them:

1. Open [the Code scanning list](https://github.com/PhinehasNarh/juice-shop/security/code-scanning).
2. Click alert #5, then **Dismiss alert**, choose **Won't fix**, type a short comment, and confirm.
3. Do the same for alert #22.

After you confirm each one, the open alert count drops, and both alerts move to Closed. If you want to undo it, open the alert again and click **Reopen alert**.

## A note on what closing does and does not do

Closing an alert is a record keeping action. It tells your team that the finding was reviewed and decided. It does not change the code. If a finding is a real risk in a real application, the right answer is to fix the code or upgrade the dependency so the alert closes as Fixed, not to dismiss it. The writeups in this folder give the exact fix for every finding so you can tell the two situations apart.
