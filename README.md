# Personal Website Monorepo

This monorepo contains a Next.js web application with AWS CDK infrastructure for deployment.

- **packages/ui/** - Next.js application (App Router)
- **infrastructure/** - AWS CDK infrastructure code (S3, CloudFront, IAM, Route 53)

## Project Structure

```
personal-website/
├── packages/            # Application packages
│   └── ui/              # Next.js app (App Router)
├── infrastructure/      # AWS CDK infrastructure
├── .github/            # GitHub workflows
└── package.json        # Root workspace configuration
```

## Development

### Prerequisites

- Node.js >= 24.0.0
- npm >= 10.8.0
- AWS CLI configured with credentials

### Setup

```bash
# Install all dependencies
npm install

# Install dependencies for specific workspace
npm install --workspace=infrastructure
npm install --workspace=packages/ui
```

### Running Locally

```bash
# Start Next.js dev server
npm run dev

# Run CDK app
npm run cdk ls

# Build all workspaces
npm run build

# Run tests
npm run test
```

### Available Scripts

| Script  | Description                         |
| ------- | ----------------------------------- |
| `build` | Build all workspaces                |
| `cdk`   | Run CDK commands                    |
| `dev`   | Start development servers           |
| `start` | Start the Next.js production server |
| `test`  | Run tests in all workspaces         |

## Deployment

The CI/CD pipeline automatically deploys on pushes to `main` or `master` branches.

### Manual CDK Deployment

```bash
cd infrastructure
npx cdk deploy
```

## Configuration

### Environment Variables

Create `.env` files in the appropriate workspace directories:

- `packages/ui/.env.local` - Next.js environment variables
- `infrastructure/.env` - CDK deployment configuration

## AWS Resources

The CDK infrastructure includes:

- S3 bucket for static assets
- CloudFront distribution for CDN
- IAM roles and policies
- Route 53 DNS configuration (if configured)
