| Feature      | Glob                       | Grep                             |
| ------------ | -------------------------- | -------------------------------- |
| Purpose      | Match **file names/paths** | Search **text inside files**     |
| Input        | File path                  | File contents                    |
| Output       | Matching file names        | Matching lines of text           |
| Pattern type | Wildcards (`*`, `?`, `**`) | Regular expressions (by default) |


Experiment with glob

List all .ts files:

ls src/*.ts

Output:

app.ts
index.ts

Recursive glob (works in shells that support **, e.g. zsh or Bash with globstar enabled):

ls src/**/*.ts

Output:

src/app.ts
src/index.ts
src/utils/math.ts
src/utils/string.ts

Try different patterns:

ls *.md
ls src/*.css
ls src/utils/*.ts
ls src/*/*.ts
Experiment with grep

Find the word TODO:

grep "TODO" src/utils/math.ts

Search every TypeScript file:

grep "import" src/**/*.ts

Search recursively:

grep -r "API_URL" .

You'll get something like:

./src/app.ts:const API_URL = "https://api.com";
Combine glob + grep

Find all .ts files containing TODO:

grep "TODO" src/**/*.ts

or recursively:

grep -r --include="*.ts" "TODO" .