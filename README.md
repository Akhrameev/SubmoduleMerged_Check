# SubmoduleMerged_Check

This repository contains a GitHub Actions workflow designed to ensure that any changes made to a specific submodule (`packages/example-dependency`) are properly merged into its `master` branch before being included in the main project. This helps to prevent accidental inclusion of unreviewed or unstable code from unmerged pull requests within the submodule.

## Workflow Description

The workflow, located in `.github/workflows/check_submodule_merged.yml`, performs the following steps:

1. **Checkout Repo:** Checks out the repository and its submodules, including the commit history needed for analysis.
2. **Fetch origin/develop:** Fetches the `develop` branch to establish comparison.
3. **Submodule Update (Force):** Initializes and updates the submodule to the correct state, ensuring any uncommitted changes are not a factor.
4. **Debug Submodule State:** Provides debug information about the state of submodules
5. **Get Submodule Path:** Extracts the path of the `packages/example-dependency` submodule from the `.gitmodules` file.
6. **Check if Submodule is Modified by Any Unmerged Commit:**  Iterates through commits on the current branch, identifying whether the submodule has been modified but the modifications have *not* yet been merged into the target branch, so check is running against the `origin/develop`.
7. **Get Commit SHA:** If the submodule is modified, it extracts the SHA of the commit from the main repository that needs to be merged into submodule's target branch.
8. **Check if Commit is Merged:** Determines if that last commit is already merged to `origin/master`. This is what makes the workflow fail if unmerged commits are found. It goes to the `packages/example-dependency` folder and executes `git merge-base --is-ancestor "$COMMIT_SHA" "$SUBMODULE_BRANCH"`.
9. **Display Merged Status:** Announces success if last commit was successfully merged to `origin/master`.
10. **Display Submodule Not Modified:** Announces success if last commit wasn't merged, but also no change was committed to `origin/master`.
11. **Fail if Not Merged:** Will interrupt the workflow if you didn't merge to `origin/master`. This behavior is what ensures your branches remain properly merged.

## Demonstration

To demonstrate the workflow, the repository contains three open pull requests, each linking to this submodule at a specific commit:

1.  **PR 2: Unreachable Commit:** The submodule is pointed to a commit that *does not exist* in this repository (e.g., a typo in the commit SHA, or a commit that was never pushed). This will cause the workflow to fail, as the commit will be unresolvable. [See example PR#2](https://github.com/Akhrameev/SubmoduleMerged_Check/pull/2)
2.  **PR 4: Merged Commit:** The submodule is pointed to a commit that *is* present in this repository and has already been merged into the `master` branch. The workflow should succeed in this case. [See example PR#4](https://github.com/Akhrameev/SubmoduleMerged_Check/pull/4)
3.  **PR 5: Unmerged Commit:** The submodule is pointed to a commit that *is* present in this repository *but has not been merged* into the `master` branch (i.e., the commit exists on a feature branch but has not been pulled into `master`). The workflow should fail in this case, indicating that the changes need to be merged first. [See example PR#5](https://github.com/Akhrameev/SubmoduleMerged_Check/pull/5)

## Configuration

* **Target Branch:** The workflow currently runs on pushes to the `develop` and `main` branches. You can adjust this in the `on:` section of the workflow file:

    ```yaml
    on:
      push:
        branches: [develop, main]
    ```

* **Submodule Branch:**  The target branch to check against in the submodule is set on `${{ github.event.inputs.submodule_branch || 'origin/master' }}`. This can be tweaked to use some another if `origin/master` is not suitable.

* **Secrets:** This workflow relies on the built-in `GITHUB_TOKEN` secret, which is automatically provided by GitHub Actions. This token is used for authenticating with Git to fetch the repository and its submodules.
Ensure the workflow has **read** permissions on the contents and workflows tabs on repo *and* *write* in creation and reviews actions to access this workflow without failures. You can change it on "Settings" tab, then search for "Actions", and then, at "General" tab, set a scope
**Workflow permissions**. You will need to set it read and write settings, at code, workflow, actions and pull request, and save this settings.

*IMPORTANT* Without correct configuration of github key on main settings, and also read write to repository, workflows are unrunnable

## Troubleshooting

* **Workflow Fails with "refusing to fetch into branch" Error:** This error occurs when trying to fetch or merge into the currently checked-out branch. If you have some issues, consider adding `"git fetch origin develop:refs/remotes/origin/develop"` as step on code.

* **Changes on Workflow Doesn't Take Effect**: After merge changes on workflow on `origin/master`, it may not be reflecting due to "GitHub Actions is failing to trigger automatically after a code push" - run a commit in all files on this folder or test this to make workflow runs again.

## Contributing

Feel free to submit pull requests to improve the workflow or add new features.
