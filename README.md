# Notify Target Repositories After Workflow Completion

This repository is configured to notify multiple target repositories whenever a workflow completes on the `main` branch. This is done by sending a `repository_dispatch` event from the source repository to the target repositories using GitHub Actions and the GitHub REST API.

## How It Works

1. **Source Repository**: When a push is made to the `main` branch of the source repository, the workflow triggers.
2. **Target Repositories**: The workflow in the source repository sends `repository_dispatch` events to one or more target repositories (`listener-repo1`, `listener-repo2`, etc.).
3. **Target Workflow Triggering**: Each target repository listens for the `repository_dispatch` event and runs its workflow accordingly.

### Workflow Overview

- **Trigger**: The workflow is triggered when a push is made to the `main` branch in the source repository.
- **Actions**:
  - The source repository's workflow checks out the code.
  - It sends a notification (via `curl` and GitHub's REST API) to multiple target repositories.
  - Each target repository is notified via the `repository_dispatch` event.

---

## Workflow Setup (Source Repository)

The following is a detailed explanation of the workflow configured in the source repository (`Notify Other Repos After Workflow Completion`):

### Prerequisites

1. **Personal Access Token (PAT)**:  
   You need to create a GitHub Personal Access Token (PAT) to authenticate the API requests. The token should have `repo` and `workflow` scopes to allow triggering repository dispatch events.

   - Go to [GitHub Settings > Developer Settings > Personal Access Tokens](https://github.com/settings/tokens).
   - Generate a new token with `repo` and `workflow` scopes.
   - Store this token as a secret named `PERSONAL_ACCESS_TOKEN` in the source repository's **Secrets**.
     - Navigate to **Settings** > **Secrets and variables** > **Actions** > **New repository secret**.
     - Add the token as `PERSONAL_ACCESS_TOKEN`.

2. **Target Repositories**:  
   The target repositories should have workflows that listen for the `repository_dispatch` event.

---

### Workflow File for Source Repository

In the source repository, the following GitHub Actions workflow (`.github/workflows/notify.yml`) is configured to trigger the target repositories when a push is made to the `main` branch.

### Explanation of Workflow Steps:

1. **Trigger**:
   - The workflow triggers on any push to the `main` branch.
2. **Checkout Code** (`actions/checkout@v3`):

   - This step uses the `actions/checkout@v3` action to checkout the code from the source repository. This isn't necessary for dispatching the events, but it's included to give access to the repository if needed for future tasks.

3. **Notify First Target Repository**:

   - This step sends a `repository_dispatch` event to the first target repository (`listener-repo1`).
   - The `curl` command is used to send a POST request to the GitHub API at `https://api.github.com/repos/MahmoudAbuzeed/listener-repo1/dispatches`.
   - The event payload includes the event type (`workflow_completed`) and the `source_repo` (which is the name of the repository triggering the event).

4. **Notify Second Target Repository**:

   - Similar to the first target repository, this step sends a `repository_dispatch` event to the second target repository (`listener-repo2`).
   - The same payload structure is used, notifying that the workflow in the source repository has been completed.

5. **Authentication**:
   - The API calls are authenticated using the Personal Access Token (`PERSONAL_ACCESS_TOKEN`), which is stored as a secret in the repository.

## GitHub API Reference

This setup relies on GitHub's REST API to trigger workflows in other repositories. The following endpoints are used:

- **Dispatch Events API**:  
  This API triggers workflows in the target repositories:

  ```
  POST /repos/{owner}/{repo}/dispatches
  ```

  For more details, refer to the [GitHub REST API documentation](https://docs.github.com/en/rest/reference/repos#create-a-repository-dispatch-event).

---

## Security Considerations

- **Secrets Management**:  
  The Personal Access Token (`PERSONAL_ACCESS_TOKEN`) is securely stored in the GitHub repository as a secret. This token should have appropriate scopes (`repo`, `workflow`) to interact with the target repositories and trigger the dispatch event.

- **Limited Scopes**:  
  Make sure the token is scoped appropriately and does not have unnecessary permissions.

---

## Conclusion

This setup allows you to notify multiple repositories when a workflow completes in the source repository, using the `repository_dispatch` event and GitHub Actions. This approach is scalable and can be easily extended by adding more target repositories in the source workflow.

Feel free to customize this further based on your specific use case
