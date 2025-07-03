# ticket3470295

This repository tests the idea of having a template that is capable of modifying the Actions workflows of itself, and repositories created from it.

To utilise this functionality, one needs to:

  1. Create a token (such as a Personal Access Token (PAT)) that has access to the Organization, and has the repository permissions:
    - READ `metadata`
    - READ/WRITE `code`
    - READ/WRITE `workflows`
  2. Place the token in the Organization (or Repository) secrets;

In this repository, the token is in an Organization secret called `WORKFLOW_TOKEN`.

This overrides the `GITHUB_TOKEN` in the checkout process:

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v3
    # This will override the GITHUB_TOKEN (it's stored in .git so using 'env:' won't work?)
    with:
      token: ${{ secrets.WORKFLOW_TOKEN }}
```

This then allows the workflow to:

  1. Modify the repository contents:

    ```yaml
    - name: Run template rendering script
      run: |
        cd .github/workflows
        python render.py
        cd -
    
    - name: Commit changes
      run: |
        git config --local user.email "github-actions[bot]@users.noreply.github.com"
        git config --local user.name "github-actions[bot]"
        git add .
        git commit -m "Render templates and replace placeholders" || echo "No changes to commit"
    ```
  
  2. Remove itself:

    ```yaml
    - name: Remove workflow file
      run: |
        # Remove the workflow file itself
        git config --local user.email "github-actions[bot]@users.noreply.github.com"
        git config --local user.name "github-actions[bot]"
        git rm .github/workflows/render-template.yml .github/workflows/render.py
        git commit -m "Remove one-time workflow file" || echo "No changes to commit"
    ```
  
  3. Push the changes to the (current or new) repository:

    ```yaml
    - name: Push changes
      run: |
        # Push the changes
        git push
    ```

In this example, we are choosing to only trigger on repository creation:

  ```yaml
  repository_dispatch:
    # This allows triggering via repo creation webhook if configured
    types: [repository-created]
  ```

And only modify repositories that are _based_ off this template, not the
template itself:

  ```yaml
  render-templates:
    ## Only run on repositories who's names don't end in '-template'
    #if: ${{ ! endsWith(github.repository, '-template') }}

    # Only run if this isn't a template rpeository
    if: ${{ ! github.event.repository.is_template }}
  ```

----
[//]: # ( vim: set ts=4 sw=4 et cindent tw=80 ai si syn=markdown ft=markdown: )
