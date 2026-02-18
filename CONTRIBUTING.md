# 🤝 Contributing to IONOS AI Model Hub + n8n Integration

We welcome contributions from the community! This guide will help you get started.

## 🚀 Quick Start

1. **Fork the repository**
2. **Clone your fork**
   ```bash
   git clone https://github.com/your-username/ionos-n8n-ai-hub.git
   cd ionos-n8n-ai-hub
   ```
3. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Make your changes**
5. **Test your changes**
6. **Submit a pull request**

## 🎯 How to Contribute

### 📝 Documentation Improvements
- Fix typos or improve clarity
- Add new examples or use cases
- Update outdated information
- Translate documentation to other languages

### 🔄 Workflow Examples
- Create new n8n workflow templates
- Improve existing workflows
- Add error handling examples
- Optimize performance patterns

### 🐛 Bug Reports
- Use the GitHub Issues template
- Include detailed reproduction steps
- Provide environment information
- Add relevant logs or screenshots

### ✨ Feature Requests
- Describe the problem you're trying to solve
- Explain your proposed solution
- Consider implementation complexity
- Discuss with maintainers first for large changes

## 📋 Contribution Guidelines

### Code Style
- Use clear, descriptive names for workflows and variables
- Add comments for complex logic
- Follow existing patterns and conventions
- Keep workflows simple and focused

### Documentation Style
- Use clear, concise language
- Include practical examples
- Add troubleshooting sections
- Use emoji for visual structure (sparingly)

### Testing
- Test all workflow examples before submitting
- Include test data or mock responses
- Document any prerequisites
- Verify compatibility with latest n8n version

## 🔧 Development Setup

### Prerequisites
- IONOS Cloud account with AI Model Hub access
- n8n instance (Docker recommended)
- Git and your preferred text editor
- Basic understanding of APIs and workflows

### Environment Setup
```bash
# Clone the repository
git clone https://github.com/your-username/ionos-n8n-ai-hub.git
cd ionos-n8n-ai-hub

# Set up your IONOS API key
export IONOS_API_KEY="your_jwt_token_here"

# Start n8n for testing
docker run -d --name n8n-dev \
  -p 5678:5678 \
  -v n8n_dev_data:/home/node/.n8n \
  -e IONOS_API_KEY="$IONOS_API_KEY" \
  n8nio/n8n:latest
```

### Testing Your Changes
```bash
# Test API connectivity
curl -H "Authorization: Bearer $IONOS_API_KEY" \
     https://openai.inference.de-txl.ionos.com/v1/models

# Import and test workflows in n8n
# Navigate to http://localhost:5678
# Import your workflow JSON files
# Execute and verify results
```

## 🏷️ Pull Request Process

### Before Submitting
- [ ] Test your changes thoroughly
- [ ] Update documentation if needed
- [ ] Follow the existing code style
- [ ] Add your changes to the changelog (if applicable)

### PR Template
```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Breaking change

## Testing
- [ ] Tested with n8n version X.X.X
- [ ] Verified with IONOS AI Model Hub
- [ ] Added/updated examples
- [ ] Documentation updated

## Screenshots (if applicable)
Add screenshots of n8n workflows or API responses.

## Checklist
- [ ] My code follows the project's style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have commented my code where necessary
- [ ] I have made corresponding changes to documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective
- [ ] New and existing tests pass
```

## 🎯 Areas We Need Help With

### High Priority
- [ ] **Performance optimization** workflows and patterns
- [ ] **Multi-language support** (German, French, Spanish)
- [ ] **Advanced RAG patterns** with metadata filtering
- [ ] **Error handling** examples and best practices
- [ ] **Cost optimization** strategies and monitoring

### Medium Priority
- [ ] **Integration examples** with other tools (Zapier, Make.com)
- [ ] **Video tutorials** for complex workflows
- [ ] **Testing frameworks** for n8n workflows
- [ ] **Monitoring dashboards** for IONOS API usage

### Low Priority
- [ ] **UI themes** for better workflow visualization
- [ ] **Workflow templates** for specific industries
- [ ] **Migration guides** from other AI platforms

## 🌟 Recognition

Contributors will be:
- Added to our README contributors section
- Mentioned in release notes for significant contributions
- Given priority support for their own issues
- Invited to join our contributor Discord channel (coming soon)

## 📞 Getting Help

### Questions About Contributing?
- 💬 Open a [Discussion](https://github.com/your-username/ionos-n8n-ai-hub/discussions)
- 📧 Email: support@360foresight.info
- 🐛 Create an [Issue](https://github.com/your-username/ionos-n8n-ai-hub/issues) for bugs

### Community Guidelines
- **Be respectful** and inclusive
- **Help others** learn and succeed
- **Share knowledge** and best practices
- **Provide constructive feedback**
- **Follow the code of conduct**

## 📊 Contribution Stats

We track and celebrate contributions:
- Number of workflows contributed
- Documentation improvements
- Bug fixes and issues resolved
- Community help and support

## 🎉 First-Time Contributors

New to open source? We'd love to help you make your first contribution!

### Good First Issues
Look for issues labeled:
- `good first issue` - Perfect for beginners
- `documentation` - Documentation improvements
- `help wanted` - We need community help
- `beginner friendly` - Easy to get started

### Mentorship
- We provide guidance for complex contributions
- Code reviews include learning opportunities
- We're patient with questions and learning

## 📝 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to the IONOS AI Model Hub + n8n community!** 🙏

Every contribution, no matter how small, helps make this project better for everyone.