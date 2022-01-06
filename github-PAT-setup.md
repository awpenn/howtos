# Setting up git to use PAT rather than password
- create PAT: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
- in repo, run following command
```
git remote set-url origin https://[username]:[token from PAT generation]@github.com/[username]/[repository].git
```
- should be able to now do things like `git push origin main` without password prompt