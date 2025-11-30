---
name: sfs-readme-gen
description: Generate comprehensive README.md files with SFS branding, badges, installation instructions, and project documentation
---

# SFS README Generator Skill

Generate professional, comprehensive README.md files following SmartFlow Systems documentation standards.

## README Template

````markdown
# [Project Name]

[![CI/CD](https://github.com/smartflow-systems/[REPO]/actions/workflows/sfs-ci-deploy.yml/badge.svg)](https://github.com/smartflow-systems/[REPO]/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

[Brief project description - 1-2 sentences]

Part of the [SmartFlow Systems](https://github.com/smartflow-systems) ecosystem.

## Features

- Feature 1
- Feature 2
- Feature 3

## Tech Stack

- **Frontend:** React 18, TypeScript, Tailwind CSS
- **Backend:** Node.js, Express
- **Database:** PostgreSQL, Prisma ORM
- **Deployment:** Replit, GitHub Actions

## Installation

```bash
git clone https://github.com/smartflow-systems/[REPO].git
cd [REPO]
npm install
cp .env.example .env
# Configure environment variables
npm run dev
```

## Environment Variables

```bash
PORT=5000
DATABASE_URL=postgresql://...
JWT_SECRET=your-secret-here
```

## Development

```bash
npm run dev      # Start development server
npm run build    # Build for production
npm run test     # Run tests
npm run lint     # Lint code
```

## API Endpoints

### Health Check
```
GET /health
Response: {"ok": true}
```

## Deployment

Automatically deploys to Replit on push to `main` branch via GitHub Actions.

## Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## License

MIT License - see LICENSE file for details.

## SmartFlow Systems

This project is part of the SmartFlow Systems ecosystem:
- [Main Site](https://github.com/smartflow-systems/SmartFlowSite)
- [Documentation](https://github.com/smartflow-systems/sfs-knowledge-base)
- [All Projects](https://github.com/orgs/smartflow-systems/repositories)

## Support

For issues and questions, please open a GitHub issue.

---

Built with ❤️ by SmartFlow Systems
````

## Usage

When invoked, scan the project to:
1. Detect project type and tech stack
2. Find existing features
3. Extract environment variables
4. Generate appropriate README sections
5. Add SFS branding and links
6. Include CI/CD badges

## Completion Checklist

- [ ] Project description added
- [ ] Feature list populated
- [ ] Tech stack documented
- [ ] Installation instructions complete
- [ ] Environment variables documented
- [ ] API endpoints listed
- [ ] SFS badges and links added
- [ ] Contributing guidelines included
