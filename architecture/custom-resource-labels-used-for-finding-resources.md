# Custom Resource Labels Used for Finding Resources

## Overview

This document provides a comprehensive reference of custom resource labels used throughout the Konflux UI codebase to query, filter, and find custom resources. These labels enable efficient resource discovery and relationship tracking across different components of the platform.

## Label Reference Table

| **Label Key** | **Purpose/Usage** | **Resource Types** | **Primary Usage Pattern** |
|---------------|------------------|-------------------|---------------------------|
| **Application & Component Labels** |
| `appstudio.openshift.io/application` | Find resources by application name | PipelineRuns, Releases, Snapshots, Components | Used in `matchLabels` selectors across hooks like `useReleases`, `usePipelineRuns` |
| `appstudio.openshift.io/component` | Find resources by component name | PipelineRuns, Secrets | Used in filtering PipelineRuns by component in `filterPipelineRuns` |
| `appstudio.application` | Alternative application label (ASEB) | PipelineRuns | Used in PipelineRun filtering |
| **Pipeline & Build Labels** |
| `pipelines.appstudio.openshift.io/type` | Filter by pipeline type (build, test, etc.) | PipelineRuns | Used to distinguish between build and test pipelines |
| `pipelines.openshift.io/used-by` | Track pipeline usage | PipelineRuns | Used in PipelineRun metadata |
| `build.appstudio.redhat.com/pipeline` | Identify Enterprise Contract pipelines | PipelineRuns | Used with value `enterprise-contract` in `isResourceEnterpriseContract` |
| `build.appstudio.openshift.io/common-secret` | Find common secrets | Secrets | Used to identify shared build secrets |
| **Tekton Resource Labels** |
| `tekton.dev/pipeline` | Link to parent pipeline | PipelineRuns | Used for pipeline relationship tracking |
| `tekton.dev/pipelineRun` | Link to parent pipeline run | TaskRuns | Used in scan results queries via `useScanResults` |
| `tekton.dev/taskRun` | Link to parent task run | TaskRuns | Used for task relationship tracking |
| `tekton.dev/task` | Link to parent task | TaskRuns | Used for task relationship tracking |
| `tekton.dev/pipelineTask` | Link to pipeline task | TaskRuns | Used for pipeline task relationship tracking |
| **Git & CI/CD Labels** |
| `pipelinesascode.tekton.dev/sha` | Find resources by commit SHA | PipelineRuns | Used in commit-based filtering |
| `pipelinesascode.tekton.dev/sender` | Find resources by commit author | PipelineRuns | Used for user-based filtering |
| `pipelinesascode.tekton.dev/url-org` | Find resources by Git organization | PipelineRuns | Used for repository-based filtering |
| `pipelinesascode.tekton.dev/url-repository` | Find resources by Git repository | PipelineRuns | Used for repository-based filtering |
| `pipelinesascode.tekton.dev/git-provider` | Find resources by Git provider | PipelineRuns | Used for provider-based filtering |
| `pipelinesascode.tekton.dev/event-type` | Find resources by Git event type | PipelineRuns | Used for event-based filtering |
| `pipelinesascode.tekton.dev/pull-request` | Find resources by PR number | PipelineRuns | Used for PR-based filtering |
| **Test & Integration Labels** |
| `test.appstudio.openshift.io/scenario` | Find test pipelines by scenario | PipelineRuns | Used in `useLatestIntegrationTestPipelines` with `matchExpressions` |
| `test.appstudio.openshift.io/optional` | Find optional integration tests | IntegrationTestScenarios | Used to identify optional test scenarios |
| `test.appstudio.openshift.io/run` | Trigger test pipeline reruns | Snapshots | Used in `rerunTestPipeline` function |
| `test.appstudio.openshift.io/create-snapshot-status` | Track snapshot creation status | PipelineRuns | Used for snapshot status tracking |
| `pac.test.appstudio.openshift.io/sha` | Find test service resources by commit | PipelineRuns | Used for test service commit tracking |
| `pac.test.appstudio.openshift.io/event-type` | Find test service resources by event type | PipelineRuns | Used for test service event filtering |
| **Release & Deployment Labels** |
| `appstudio.openshift.io/snapshot` | Find resources by snapshot | PipelineRuns | Used to link resources to snapshots |
| `appstudio.openshift.io/build-pipelinerun` | Find snapshots by build pipeline run | Snapshots | Used for build-to-snapshot relationship |
| `release.appstudio.openshift.io/auto-release` | Find auto-release plans | ReleasePlans | Used to identify automatically triggered releases |
| `release.appstudio.openshift.io/standing-attribution` | Find standing attribution releases | ReleasePlans | Used for release attribution tracking |
| **Secret & Security Labels** |
| `ui.appstudio.redhat.com/secret-for` | Find secrets by UI purpose | Secrets | Used in secret selection and filtering |
| `appstudio.redhat.com/environment` | Find secrets by environment | Secrets | Used for environment-specific secrets |
| `appstudio.redhat.com/credentials` | Find credential secrets | Secrets | Used with value `scm` for SCM credentials |
| `appstudio.redhat.com/scm.host` | Find secrets by SCM host | Secrets | Used for host-specific SCM secrets |

## Key Usage Patterns

### 1. matchLabels
The most common pattern using exact label matching in resource selectors:

```typescript
selector: {
  matchLabels: {
    [PipelineRunLabel.APPLICATION]: applicationName,
    [PipelineRunLabel.PIPELINE_TYPE]: PipelineRunType.BUILD,
  },
}
```

### 2. matchExpressions
Used for complex queries with operators like `In`, `Exists`, etc.:

```typescript
selector: {
  matchExpressions: [
    {
      key: PipelineRunLabel.TEST_SERVICE_SCENARIO,
      operator: 'In',
      values: neededNames,
    },
  ],
}
```

### 3. Client-side Filtering
Labels used in client-side filtering of already-retrieved resources:

```typescript
pipelineRuns.filter((plr) => 
  plr.metadata.labels?.[PipelineRunLabel.COMPONENT]?.indexOf(name) >= 0
)
```

### 4. Tekton Results Filtering
Special filter syntax for querying Tekton Results API:

```typescript
filter: labelsToFilter({
  [TektonResourceLabel.pipelinerun]: pipelineRunName,
})
```

## Main Query Functions

- **`usePipelineRuns`**: Uses multiple labels for application, component, pipeline type, and commit filtering
- **`useReleases`**: Uses application labels for release filtering  
- **`useScanResults`**: Uses pipeline run labels for security scan results
- **`useLatestIntegrationTestPipelines`**: Uses test scenario labels with match expressions
- **`useSnapshots`**: Uses application labels for snapshot filtering

## Code Locations

### Label Constants
- `src/consts/pipelinerun.ts` - PipelineRun and snapshot labels
- `src/types/coreTekton.ts` - Tekton resource labels
- `src/types/secret.ts` - Secret-related labels
- `src/types/coreBuildService.ts` - ReleasePlan labels

### Query Utilities
- `src/k8s/k8s-utils.ts` - Label selector utilities
- `src/utils/tekton-results.ts` - Tekton Results filtering
- `src/k8s/query/utils.ts` - Query key generation

### Usage Examples
- `src/hooks/usePipelineRuns.ts` - Pipeline run queries
- `src/hooks/useReleases.ts` - Release queries
- `src/hooks/useScanResults.ts` - Security scan queries
- `src/components/Filter/utils/` - Filtering utilities

## Notes

This label system provides a comprehensive way to query and filter custom resources across the Konflux platform, enabling efficient resource discovery and relationship tracking. The labels follow consistent naming conventions with domain prefixes indicating their purpose and scope. 

