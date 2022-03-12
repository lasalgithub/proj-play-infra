# Play Infra
Play Hub infrastructure components

## Add the git hub package source
```s
OWNER=playhuborg
GH_PAT=[GitHubToken]

dotnet nuget add source --username USERNAME --password $GH_PAT --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$OWNER/index.json"
```

