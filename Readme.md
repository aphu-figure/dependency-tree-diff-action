# Dependency Tree Diff - Github Action

Simple Github Action wrapper for Jake Wharton's [Dependency Tree Diff](https://github.com/JakeWharton/dependency-tree-diff) tool.

## Usage 
The action only exposes _output_ containing the diff, so to effectively consume its output it is highly recommended to use other Github Actions to customize your experience.

#### Create Pull Request comment on dependency change   
Create `.github/workflows/dependency_diff.yml`

```yml
name: Generate dependency diff

on:
  pull_request:

jobs:
  generate-diff:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0
    - name: set up JDK 1.8
      uses: actions/setup-java@v1
      with:
          java-version: 1.8
    - name: Cache .gradle
      uses: burrunan/gradle-cache-action@v1
    - id: dependency-diff
      name: Generate dependency diff
      uses: usefulness/dependency-tree-diff-action@master
    - uses: peter-evans/find-comment@v1
      id: find_comment
      with:
        issue-number: ${{ github.event.pull_request.number }}
        body-includes: Dependency diff
    - uses: peter-evans/create-or-update-comment@v1
      if: ${{ steps.dependency-diff.outputs.text-diff != null || steps.find_comment.outputs.comment-id != null }}
      with:
        body: |
          Dependency diff (customize your message here): 
            ```diff
            ${{ steps.dependency-diff.outputs.text-diff }}
            ```
        edit-mode: replace
        comment-id: ${{ steps.find_comment.outputs.comment-id }}
        issue-number: ${{ github.event.pull_request.number }}
        token: ${{ secrets.GITHUB_TOKEN }}
```

## Customization
All inputs with their default values:
```yml
    - id: dependency-diff
      name: Generate dependency diff
      uses: usefulness/dependency-tree-diff-action@master
      with:
        configuration: 'releaseRuntimeClasspath'
        project: 'app'
        build-root-directory: .
        lib-version: '1.1.0'
```

- **`configuration`** - Selected Gradle configuration, passed to `./gradlew dependencies --configuration xxx`.
Should correspond to output artifact that is considered output of the project.  **(Required)**
- **`project`** - Gradle project which dependency tree diff should be generated for. 
Dependency diff for root projects can be configured using `project: ''`. 
 For Android projects use the one that has `com.android.application` plugin applied. **(Required)**
- **`build-root-directory`** - Relative path to folder containing gradle wrapper. 
Example usage: `build-root-directory: library` **(Required)**
- **`lib-version`** - Overrides [dependency-tree-diff](https://github.com/JakeWharton/dependency-tree-diff) dependency version _(Optional)_

<details><summary></summary>
<p>

🙏 Praise 🙏 be 🙏 to 🙏 Wharton 🙏

</p>
</details>
