name: Generate README for /database

on:
  push:
    paths:
      - 'database/**'
  workflow_dispatch:

jobs:
  generate-readme:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'

    - name: Install dependencies
      run: npm install json-to-markdown-table

    - name: Generate README.md
      run: |
        # Create the initial README content
        echo "# Red Horse" > database/README.md
        echo "## Sanctuary Massacre Orchestrations" >> database/README.md
        node -e "const { readFileSync, writeFileSync } = require('fs'); \
        const { jsonToMarkdownTable } = require('json-to-markdown-table'); \
        const smDb = JSON.parse(readFileSync('./database/sm-db.json', 'utf8')); \
        const pfDb = JSON.parse(readFileSync('./database/pf-db.json', 'utf8')); \
        const ppDb = JSON.parse(readFileSync('./database/pp-db.json', 'utf8')); \
        const smTable = jsonToMarkdownTable(smDb, { padding: 1 }); \
        const pfTable = jsonToMarkdownTable(pfDb, { padding: 1 }); \
        const ppTable = jsonToMarkdownTable(ppDb, { padding: 1 }); \
        writeFileSync('./database/README.md', `# Red Horse\n\n## Sanctuary Massacre Orchestrations\n\n${smTable}\n\n## Phantom Funeral Orchestrations\n\n${pfTable}\n\n# Black Horse\n\n## Phantom Pantry Orchestrations\n\n${ppTable}\n`, { flag: 'a' });"

    - name: Commit and push changes
      run: |
        git config --local user.name "github-actions[bot]"
        git config --local user.email "github-actions[bot]@users.noreply.github.com"
        git add database/README.md
        git commit -m "Update README.md with JSON data"
        git push
