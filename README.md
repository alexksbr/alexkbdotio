# Hugo Site with Congo Theme

This site is built with [Hugo](https://gohugo.io/) and the [Congo theme](https://github.com/jpanther/congo).

## Local Development

```bash
# Run the development server
hugo server -D

# Build the site
hugo --gc --minify
```

## Deployment

This site is configured to deploy to Netlify automatically when pushed to GitHub.

### Setup Instructions

1. Push this repository to GitHub
2. Connect your GitHub repository to Netlify
3. Netlify will automatically detect the `netlify.toml` configuration
4. Your site will build and deploy automatically

### Configuration

- Update `baseURL` in `hugo.toml` with your actual domain
- Customize the Congo theme by copying config files from `themes/congo/config/_default/` to your root `config/_default/` directory
- See [Congo documentation](https://jpanther.github.io/congo/) for theme customization options

## Theme Updates

To update the Congo theme:

```bash
git submodule update --remote --merge
```
