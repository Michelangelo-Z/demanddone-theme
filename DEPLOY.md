# DemandDone Theme Deployment Guide

## Prerequisites

Install Shopify CLI:

```bash
# macOS
brew install shopify-cli

# Or via npm
npm install -g @shopify/cli @shopify/theme
```

## Quick Deploy

### 1. Login to your Shopify store

```bash
shopify login --store demanddone.myshopify.com
```

This will open a browser window for authentication.

### 2. Preview locally (recommended first)

```bash
cd ~/Documents/Assistx/GitHub/demanddone-theme
shopify theme dev
```

This starts a local dev server. Open the preview URL to see your branded theme.

### 3. Push as unpublished theme (safe testing)

```bash
shopify theme push --unpublished
```

This creates a new theme in your Shopify admin without making it live. You can preview it on your store.

### 4. Publish when ready

```bash
# List themes to get the ID
shopify theme list

# Publish the theme
shopify theme publish --theme YOUR_THEME_ID
```

## Alternative: Upload via Shopify Admin

### 1. Package theme as ZIP

```bash
cd ~/Documents/Assistx/GitHub/demanddone-theme
zip -r demanddone-theme.zip . -x "*.git*" -x ".DS_Store"
```

### 2. Upload in Shopify Admin

1. Go to: Online Store > Themes
2. Click "Add theme" > "Upload ZIP file"
3. Select `demanddone-theme.zip`
4. Click "Publish" when ready

## Updating the Theme

### Method 1: Regenerate from preset

If you update the brand preset JSON:

```bash
# From ai-sale-store project root
npx tsx shopify-theme/scripts/build-theme.ts demanddone.default

# Copy updated files
cp shopify-theme/build/demanddone.default/config/settings_data.json ~/Documents/Assistx/GitHub/demanddone-theme/config/
cp shopify-theme/build/demanddone.default/snippets/demanddone-tokens.liquid ~/Documents/Assistx/GitHub/demanddone-theme/snippets/
cp shopify-theme/build/demanddone.default/templates/index.json ~/Documents/Assistx/GitHub/demanddone-theme/templates/

# Commit changes
cd ~/Documents/Assistx/GitHub/demanddone-theme
git add -A
git commit -m "chore: update brand tokens"

# Deploy
shopify theme push
```

### Method 2: Direct edits

For quick tweaks, edit files directly in the theme folder, then:

```bash
cd ~/Documents/Assistx/GitHub/demanddone-theme
git add -A
git commit -m "style: adjust button radius"
shopify theme push
```

## Testing Before Going Live

### A. Use Theme Preview

After pushing as unpublished:

```bash
shopify theme list
# Note the theme ID

# Get preview URL
shopify theme preview --theme YOUR_THEME_ID
```

Share this URL with stakeholders for review.

### B. Duplicate Live Theme First

1. In Shopify Admin, go to Online Store > Themes
2. Find your live theme, click "Actions" > "Duplicate"
3. Push updates to the duplicate first
4. Test thoroughly
5. Publish duplicate when ready

## Continuous Deployment (Future)

For automated deployment on git push:

```bash
# GitHub Actions workflow (example)
name: Deploy Theme
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Shopify
        env:
          SHOPIFY_CLI_THEME_TOKEN: ${{ secrets.SHOPIFY_CLI_THEME_TOKEN }}
          SHOPIFY_FLAG_STORE: demanddone.myshopify.com
        run: |
          npm install -g @shopify/cli
          shopify theme push --theme YOUR_THEME_ID
```

## Rollback

If something goes wrong:

```bash
# List themes
shopify theme list

# Pull previous version from Shopify
shopify theme pull --theme PREVIOUS_THEME_ID

# Or revert git commit
git revert HEAD
shopify theme push
```

## Troubleshooting

### "Theme not found"

```bash
shopify theme list
# Use the exact theme ID shown
shopify theme push --theme 123456789
```

### "Permission denied"

Ensure you're logged in:

```bash
shopify logout
shopify login --store demanddone.myshopify.com
```

### CSS not updating

Hard refresh browser (Cmd+Shift+R on Mac) or clear Shopify CDN cache:

```bash
shopify theme push --force
```

### Local preview not working

Check that port 9292 isn't in use:

```bash
lsof -ti:9292 | xargs kill -9
shopify theme dev
```

## Best Practices

1. **Always push unpublished first** - Test before going live
2. **Use git commits** - Track all changes
3. **Test on mobile** - Preview on real devices
4. **Monitor after deploy** - Check for console errors in browser dev tools
5. **Keep Horizon updated** - Periodically merge upstream Horizon updates

## Resources

- [Shopify CLI Docs](https://shopify.dev/docs/api/shopify-cli)
- [Horizon Theme Repo](https://github.com/Shopify/horizon)
- [Theme Architecture](https://shopify.dev/docs/storefronts/themes/architecture)
