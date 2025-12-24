# Competitors Folder Context

## What's Here

Competitive analysis of open source prediction market trading bots.

## Reference Repositories

All analyzed repositories are cloned in `../../reference/` for direct code inspection:

```bash
cd ../../reference/
ls -la  # See all cloned repos
```

## How to Analyze

When adding new competitive analysis:

1. **Clone the repo**:

   ```bash
   cd /Users/nick/src/reference
   git clone <repo-url>
   ```

2. **Launch Explore agent** to walk through codebase and understand:
   - Project structure
   - Trading strategy and logic
   - Risk management approach
   - Architecture and tech stack
   - Code quality and documentation
   - Unique innovations

3. **Write competitive analysis** in this directory following existing structure

4. **Extract learnings** - what to steal, what to avoid, what they do well

## Analysis Structure

Each analysis should cover:

- Overview and core thesis
- Strategy (how they trade)
- Risk management (how they protect capital)
- Architecture (how it's built)
- Code quality (tests, docs, maintainability)
- Unique innovations (novel approaches)
- What to learn (key takeaways)
- Philosophical differences (how they differ from us)
- Rating (1-5 stars)

## Working with Reference Repos

The repos in `../../reference/` are for:

- Direct code inspection
- Understanding implementation details
- Finding specific patterns to reference
- Learning from working examples

Don't modify them - they're read-only reference material.
