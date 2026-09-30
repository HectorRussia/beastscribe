# BOO-4: Linear and Discord integration smoke test

Use `ponkritwo/boo-4-test-linear-discord` for the first integration check.

1. Start BOO-4 in In Progress and open a ready PR targeting dev.
2. Check the Linear status sync Actions log for In Progress -> In Review.
3. Merge the test PR into dev and check In Review -> QA.
4. Stop at dev until the release is approved. No Discord message is expected yet.
5. Later merge dev into main with Create a merge commit. Check the configured
   Discord forum thread for BOO-4, while the Linear issue remains in QA.

Use Actions logs and the actual issue status as evidence; this checklist does
not itself assert that a check passed. Re-running the Discord job may resend.
