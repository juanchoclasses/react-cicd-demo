# React CI/CD Exercise with GitHub Actions

**Time Required:** ~45 minutes  
**Learning Objectives:**
- Understand a React TypeScript application with automated testing
- Implement continuous integration (CI) with GitHub Actions
- Deploy automatically to GitHub Pages (CD)
- Experience the full CI/CD pipeline
- Practice team collaboration with Git workflows

## Prerequisites
- Node.js and npm installed
- Git installed
- GitHub account
- Basic familiarity with React and command line
- (Optional) GitHub CLI (`gh`) for faster repository setup

## 📦 Getting Started

The starter application is available at: **https://github.com/juanchoclasses/react-cicd-demo/**

## Team Setup (5 minutes)  

**This is a team exercise. One team member should:**

1. **Fork the repository:**
   - Go to https://github.com/juanchoclasses/react-cicd-demo/
   - Click the **"Fork"** button in the top-right corner
   - **Important:** Keep the repository name as `react-cicd-demo` (do NOT change it)
   - This creates a copy in your GitHub account: `https://github.com/YOUR-USERNAME/react-cicd-demo/`
   
   > ⚠️ **Note:** The repository name must remain `react-cicd-demo` because the build configuration (`vite.config.ts`) uses this path for GitHub Pages deployment. If you change the name, the deployed site won't work correctly.

2. **Clone your fork locally:**
   ```bash
   # Replace YOUR-USERNAME with your actual GitHub username
   git clone https://github.com/YOUR-USERNAME/react-cicd-demo.git
   cd react-cicd-demo
   ```

3. **Invite team members as collaborators:**
   - Go to your forked repository on GitHub
   - Navigate to **Settings** → **Collaborators**
   - Click **Add people**
   - Add each team member's GitHub username
   - **Important:** Add the professor: **sillyfunnypedro**

**All other team members should:**
- Accept the collaboration invitation (check email or GitHub notifications)
- Clone the team's forked repository:
  ```bash
  git clone https://github.com/TEAM-LEAD-USERNAME/react-cicd-demo.git
  cd react-cicd-demo
  ```

## Part 1: Understanding the Provided Application (5 minutes)

**The application has been created for you.** Take a few minutes to explore the files and understand the structure.

### Project Structure
```
react-cicd-demo/
├── src/
│   ├── App.tsx          # Main component with counter logic (TypeScript)
│   ├── App.test.tsx     # Test suite for App component (TypeScript)
│   ├── main.tsx         # React entry point (TypeScript)
│   ├── setupTests.ts    # Jest setup with jest-dom matchers
│   └── App.css          # Styling
├── index.html           # HTML template
├── package.json         # Dependencies and scripts
├── tsconfig.json        # TypeScript configuration
├── tsconfig.node.json   # TypeScript config for Node
├── vite.config.ts       # Vite bundler configuration (TypeScript)
├── jest.config.js       # Jest test configuration
├── .babelrc            # Babel transpiler configuration
└── .gitignore          # Git ignore rules
```

### Key Features
- **TypeScript** for type safety and better developer experience
- **Simple counter app** with increment and reset buttons
- **Comprehensive test suite** using React Testing Library
- **Vite** for fast development and building
- **Jest** configured for testing TypeScript

### Install Dependencies and Test Locally
```bash
# You should already be in the react-cicd-demo directory
# If not: cd react-cicd-demo

# Install all dependencies
npm install

# Run tests (should all pass)
npm test

# Start development server (optional)
npm run dev
```

If you run the dev server, visit `http://localhost:5173` to see your app running.

**Checkpoint:** Make sure all tests pass before continuing!

## Part 2: Verify Your Setup (2 minutes)

Before configuring CI/CD, let's verify everything is working:

1. **Check your repository on GitHub:**
   - Visit your forked repository: `https://github.com/YOUR-USERNAME/react-cicd-demo/`
   - Confirm all files are present (especially `.gitignore`, `.babelrc`, and `.github/workflows/`)

2. **Verify git remote:**
   ```bash
   # Should show your fork, not the original repo
   git remote -v
   ```

3. **All teammates should have cloned the fork and run:**
   ```bash
   npm install
   npm test
   ```

**You're now ready to set up the CI/CD pipeline!**

## Part 3: Configure GitHub Actions for CI/CD (15 minutes)

### Step 1: Create GitHub Actions Workflow
Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Add .nojekyll
        run: touch ./dist/.nojekyll

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Step 2: Commit and Push Workflow
```bash
git add .github/
git commit -m "Add CI/CD workflow"
git push
```

### Step 3: Configure Repository Permissions
**Important:** You must configure these settings for the workflow to work:

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Actions** → **General**
3. Scroll down to **Workflow permissions**
4. Select **"Read and write permissions"**
5. Check **"Allow GitHub Actions to create and approve pull requests"**
6. Click **Save**

### Step 4: Enable GitHub Pages
1. Go to your repository on GitHub
2. Click **Settings** → **Pages**
3. Under **Build and deployment** → **Source**
4. Select **GitHub Actions** (NOT "Deploy from a branch")
5. Click **Save**

### Step 5: Verify the Pipeline
1. Go to the **Actions** tab in your repository
2. You should see your workflow running
3. Watch as it runs tests, builds, and deploys
4. Once complete, your site will be available at: `https://YOUR-USERNAME.github.io/react-cicd-demo/`

## Part 4: Test the CI/CD Pipeline (15 minutes)

### Exercise 1: Make a Feature Change
1. Create a new branch:
   ```bash
   git checkout -b feature/add-decrement
   ```

2. Modify `src/App.tsx` to add a decrement button:
   ```typescript
   <button onClick={() => setCount(count - 1)}>
     Decrement
   </button>
   ```

3. Add a test in `src/App.test.tsx`:
   ```typescript
   test('decrement button decreases count', () => {
     render(<App />)
     const incrementButton = screen.getByText(/Increment/i)
     const decrementButton = screen.getByText(/Decrement/i)
     
     fireEvent.click(incrementButton)
     fireEvent.click(incrementButton)
     fireEvent.click(decrementButton)
     
     const count = screen.getByText(/Click count: 1/i)
     expect(count).toBeInTheDocument()
   })
   ```

4. Commit and push:
   ```bash
   git add .
   git commit -m "Add decrement button"
   git push origin feature/add-decrement
   ```

5. **Create a Pull Request on GitHub:**
   
   After pushing, you'll see a message in your terminal with a link, or:
   
   - Go to your repository on GitHub
   - You'll see a **yellow banner** at the top saying "**feature/add-decrement** had recent pushes" with a green **"Compare & pull request"** button
   - Click **"Compare & pull request"**
   
   **Alternative method** (if you don't see the banner):
   - Click the **"Pull requests"** tab at the top
   - Click the green **"New pull request"** button
   - Select your branch from the "compare" dropdown
   
   Fill in the PR details:
   - **Title**: "Add decrement button" (auto-filled from commit message)
   - **Description**: Add a brief description like "Adds a decrement button to decrease the counter"
   - Click **"Create pull request"**

6. **Observe**: GitHub Actions runs tests automatically on your PR
   - Look for the "Checks" section below your PR description
   - You'll see "CI/CD Pipeline" running
   - Wait for it to complete (should show a green checkmark ✓)

7. **Merge the PR after tests pass:**
   - Once tests pass, click the **"Merge pull request"** button
   - Click **"Confirm merge"**
   - Optionally, click **"Delete branch"** to clean up

8. **Observe**: The deploy job runs automatically
   - Go to the **Actions** tab
   - You'll see the deployment running for the main branch
   - Wait for it to complete

9. Visit your GitHub Pages site to see the changes live:
   - `https://YOUR-USERNAME.github.io/react-cicd-demo/`
   - You should now see the decrement button!

### Exercise 2: Break the Build (Intentionally)
1. Create another branch:
   ```bash
   git checkout main
   git pull
   git checkout -b feature/broken-test
   ```

2. Modify `src/App.tsx` to change the heading text:
   ```typescript
   <h1>React CI/CD Updated</h1>
   ```

3. Don't update the test (so it will fail)

4. Push and create a PR:
   ```bash
   git add .
   git commit -m "Update heading"
   git push origin feature/broken-test
   ```

5. **Observe**: Tests fail in the PR, preventing merge
   - Create the PR the same way as Exercise 1
   - Notice the **red X** instead of a green checkmark
   - Click on "Details" to see which test failed
   - GitHub won't let you merge (if branch protection is enabled)

6. Fix the test in `src/App.test.tsx`:
   ```typescript
   const heading = screen.getByText(/React CI\/CD Updated/i)
   ```

7. Commit and push the fix:
   ```bash
   git add .
   git commit -m "Fix test for updated heading"
   git push origin feature/broken-test
   ```
   - **Observe**: Tests automatically re-run and should now pass ✓

8. Merge the PR and observe automatic deployment

## Discussion Questions

1. **What is the purpose of the `needs: test` line in the deploy job?**
   - Answer: It ensures deployment only happens if tests pass

2. **Why do we run tests on pull requests before merging?**
   - Answer: To catch bugs before they reach the main branch

3. **What happens if you push broken code directly to main?**
   - Answer: Tests will fail but code is already merged; deployment may fail

4. **How could you add code linting to this pipeline?**
   - Answer: Add ESLint and a linting step before tests

5. **What are the benefits of automated deployment?**
   - Answer: Consistency, speed, reduced human error, immediate feedback

## Extension Ideas

- Add ESLint for code quality checks
- Add code coverage reporting
- Set up multiple environments (staging, production)
- Add Slack or email notifications for build failures
- Implement semantic versioning and automated releases
- Add security scanning with tools like Snyk or Dependabot

## Troubleshooting

**Tests fail locally but not in CI:**
- Check Node.js versions match
- Ensure all dependencies are in package.json

**Deployment fails with 403 Permission Error:**
- Go to **Settings** → **Actions** → **General** → **Workflow permissions**
- Select **"Read and write permissions"**
- This is required for the deployment action to work

**Deployment fails:**
- Verify GitHub Pages is enabled in **Settings** → **Pages**
- Ensure **Source** is set to **"GitHub Actions"** (not "Deploy from a branch")
- Check that base path in `vite.config.ts` matches repo name
- Make sure workflow permissions are configured correctly (see above)

**Site shows 404 or assets fail to load:**
- Wait a few minutes for GitHub Pages to build and deploy
- **Most common issue:** Verify your repository is named exactly `react-cicd-demo` (the base path in `vite.config.ts` depends on this)
- If you renamed the repository, either:
  - Rename it back to `react-cicd-demo`, OR
  - Update the `base` path in `vite.config.ts` to match your repo name (e.g., `base: '/your-repo-name/'`)
- Verify the deployment completed successfully in the **Actions** tab
- Ensure `.nojekyll` file is being created in the build (workflow does this automatically)