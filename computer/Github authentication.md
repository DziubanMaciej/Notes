parentTopic:: [[Computer]]

# Github authentication

With personal access token:
1. Go to https://github.com/settings/tokens and generate a token
2. If organization SSO is enabled, click on "Configure SSO" and given this token access to the organization
3. Save the token, you will not see it again (can regenerate later).
4. During git clone input any username (e.g. "x", it doesn't matter) and paste the token as the password
5. Git will remember this token on Windows. You can clear it with `git config --global --unset credential.helper`
