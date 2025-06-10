# Git Integration in BetterGPT

## Overview

Our Git integration provides version control features, code review capabilities, and collaboration tools directly within the BetterGPT interface. It combines local Git operations with GitHub API integration for a seamless development experience.

## Core Features

| Feature | Description | Priority |
|---------|-------------|----------|
| Version Control | Basic Git operations and history | P0 |
| Code Review | PR review and inline comments | P0 |
| Diff Viewing | Visual diff comparison | P1 |
| Commit Suggestions | AI-powered commit messages | P2 |

## Implementation

### 1. Git Service

```typescript
// src/services/GitService.ts
import simpleGit, { SimpleGit } from 'simple-git';
import { Octokit } from '@octokit/rest';

export class GitService {
  private git: SimpleGit;
  private octokit: Octokit;

  constructor() {
    this.git = simpleGit();
    this.octokit = new Octokit({
      auth: process.env.GITHUB_TOKEN
    });
  }

  async getFileHistory(filePath: string): Promise<GitLogLine[]> {
    const log = await this.git.log({
      file: filePath,
      maxCount: 50
    });
    
    return log.all.map(commit => ({
      hash: commit.hash,
      date: commit.date,
      message: commit.message,
      author: commit.author_name
    }));
  }

  async getDiff(
    filePath: string,
    oldRef: string,
    newRef: string
  ): Promise<string> {
    return this.git.diff([oldRef, newRef, '--', filePath]);
  }

  async suggestCommitMessage(
    diff: string
  ): Promise<string> {
    const openai = new OpenAIService();
    return openai.generateCommitMessage(diff);
  }
}
```

### 2. GitHub Integration

```typescript
// src/services/GitHubService.ts
export class GitHubService {
  private octokit: Octokit;

  async createPullRequest(
    options: CreatePROptions
  ): Promise<PullRequest> {
    return this.octokit.pulls.create({
      owner: options.owner,
      repo: options.repo,
      title: options.title,
      head: options.branch,
      base: options.targetBranch,
      body: options.description
    });
  }

  async reviewPullRequest(
    pr: number,
    review: ReviewInput
  ): Promise<void> {
    await this.octokit.pulls.createReview({
      owner: review.owner,
      repo: review.repo,
      pull_number: pr,
      event: review.event,
      body: review.body,
      comments: review.comments
    });
  }

  async getReviewComments(
    pr: number
  ): Promise<Comment[]> {
    const { data } = await this.octokit.pulls.listReviewComments({
      owner: this.owner,
      repo: this.repo,
      pull_number: pr
    });
    
    return data.map(this.formatComment);
  }
}
```

### 3. Monaco Editor Integration

```typescript
// src/components/DiffViewer.tsx
import { DiffEditor } from '@monaco-editor/react';

export const DiffViewer = ({ 
  oldContent,
  newContent,
  language
}) => {
  return (
    <DiffEditor
      height="500px"
      language={language}
      original={oldContent}
      modified={newContent}
      options={{
        renderSideBySide: true,
        enableSplitViewResizing: true,
        originalEditable: false
      }}
    />
  );
};
```

### 4. Review Component

```typescript
// src/components/PullRequestReview.tsx
export const PullRequestReview = ({ pr }: { pr: PullRequest }) => {
  const [comments, setComments] = useState<Comment[]>([]);
  const github = new GitHubService();

  const addComment = async (
    path: string,
    position: number,
    body: string
  ) => {
    await github.createReviewComment({
      owner: pr.owner,
      repo: pr.repo,
      pull_number: pr.number,
      body,
      path,
      position
    });
    
    refreshComments();
  };

  return (
    <div>
      <DiffViewer
        oldContent={pr.base}
        newContent={pr.head}
        language={pr.language}
        onLineClick={position => {
          // Show comment dialog
        }}
      />
      <CommentList comments={comments} />
    </div>
  );
};
```

## Required Dependencies

```json
{
  "dependencies": {
    "simple-git": "^3.20.0",
    "@octokit/rest": "^19.0.0",
    "@monaco-editor/react": "^4.6.0",
    "diff": "^5.1.0"
  }
}
```

## Environment Configuration

```env
GITHUB_TOKEN=your_github_token
GITHUB_WEBHOOK_SECRET=your_webhook_secret
REPO_PATH=/path/to/repo
DEFAULT_BRANCH=main
```

## Webhook Handler

```typescript
// src/api/github/webhook.ts
import { createHmac } from 'crypto';

export async function handleWebhook(
  req: Request,
  res: Response
) {
  const signature = req.headers['x-hub-signature-256'];
  if (!verifySignature(signature, req.body)) {
    return res.status(401).send('Invalid signature');
  }

  const event = req.headers['x-github-event'];
  switch (event) {
    case 'pull_request':
      await handlePullRequest(req.body);
      break;
    case 'push':
      await handlePush(req.body);
      break;
    // Handle other events...
  }

  res.status(200).send('OK');
}
```

## Best Practices

1. **Security**
   - Secure token storage
   - Webhook signature verification
   - Access control checks

2. **Performance**
   - Lazy load Git history
   - Cache common operations
   - Optimize large diffs

3. **UX Guidelines**
   - Clear diff visualization
   - Intuitive review flow
   - Keyboard shortcuts

## Error Handling

```typescript
class GitError extends Error {
  constructor(
    message: string,
    public code: string,
    public command?: string
  ) {
    super(message);
    this.name = 'GitError';
  }
}

function handleGitError(error: any) {
  if (error.message.includes('not a git repository')) {
    throw new GitError(
      'Not a Git repository',
      'NOT_GIT_REPO'
    );
  }
  // Handle other common errors...
}
```

## Monitoring

```typescript
class GitMetrics {
  async trackOperation(
    operation: string,
    duration: number,
    success: boolean
  ) {
    await db.insert(gitOperationLogs).values({
      operation,
      duration,
      success,
      timestamp: new Date()
    });
  }

  async getOperationStats(): Promise<OperationStats> {
    return db
      .select({
        avgDuration: sql`avg(duration)`,
        successRate: sql`sum(case when success then 1 else 0 end)::float / count(*)`,
        totalOperations: sql`count(*)`
      })
      .from(gitOperationLogs);
  }
}
```

## Future Improvements

1. **Advanced Features**
   - Branch visualization
   - Merge conflict resolution
   - Interactive rebase UI

2. **AI Integration**
   - Smart code review suggestions
   - Automated PR summaries
   - Commit message generation

3. **Collaboration**
   - Real-time review comments
   - Team review assignments
   - Review status tracking 