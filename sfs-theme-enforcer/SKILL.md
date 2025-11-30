---
name: sfs-theme-enforcer
description: Apply or update the SmartFlow Systems brown/black/gold theme to any project with Tailwind CSS configuration, color palette, and brand assets
---

# SFS Theme Enforcer Skill

This skill applies the signature SmartFlow Systems brown/black/gold aesthetic to any project, ensuring brand consistency across the entire SFS ecosystem.

## When to Use This Skill

Invoke this skill when:
- Starting a new SFS project that needs branding
- Updating an existing project to match SFS theme standards
- Converting a client project to SFS white-label branding
- Fixing theme inconsistencies across projects
- Refreshing outdated brand styling

## What This Skill Does

### 1. Tailwind CSS Configuration
- Install and configure Tailwind CSS if not present
- Add SFS color palette to `tailwind.config.js`
- Set up custom theme extensions
- Configure typography and spacing

### 2. Color Palette Application
- Primary brown shades (50-900)
- Signature black (#0a0908)
- Gold accents (300-600)
- Configure semantic colors (primary, secondary, accent)

### 3. Component Styling
- Apply theme to buttons, cards, navigation
- Set up hover states with gold accents
- Configure focus states and accessibility colors
- Style form inputs and interactive elements

### 4. Typography System
- Configure font families
- Set up heading styles
- Define text color hierarchy
- Apply consistent letter spacing and line heights

### 5. Brand Assets Integration
- Reference SFS logo locations
- Set up favicon paths
- Configure brand imagery
- Add social media preview images

### 6. Dark Mode Support (Optional)
- Configure dark mode variants
- Set up theme toggle if needed
- Apply appropriate color inversions

## SFS Color System

### Primary Brown Scale
```javascript
brown: {
  50: '#f8f6f4',   // Lightest - backgrounds
  100: '#e8e2dc',  // Very light - subtle backgrounds
  200: '#d4c4b8',  // Light - borders, dividers
  300: '#bfa490',  // Medium-light - disabled states
  400: '#a8846e',  // Medium - secondary text
  500: '#8b6f47',  // Primary brown - main brand color
  600: '#6b5435',  // Dark - primary text, headings
  700: '#4a3a24',  // Darker - emphasis
  800: '#2d2416',  // Very dark - strong contrast
  900: '#1a140c',  // Darkest - maximum contrast
}
```

### Signature Black
```javascript
black: '#0a0908'  // SFS signature black
```

### Gold Accents
```javascript
gold: {
  300: '#ffd700',  // Light gold - highlights
  400: '#ffcc00',  // Medium gold - hover states
  500: '#d4af37',  // Classic gold - primary accent
  600: '#b8941e',  // Dark gold - pressed states
}
```

### Semantic Colors
```javascript
primary: sfs.brown[500]
secondary: sfs.brown[300]
accent: sfs.gold[500]
background: sfs.brown[50]
surface: white
text: {
  primary: sfs.brown[800]
  secondary: sfs.brown[600]
  disabled: sfs.brown[300]
}
```

## Complete Tailwind Config

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './src/**/*.{js,jsx,ts,tsx}',
    './public/index.html',
  ],
  theme: {
    extend: {
      colors: {
        sfs: {
          brown: {
            50: '#f8f6f4',
            100: '#e8e2dc',
            200: '#d4c4b8',
            300: '#bfa490',
            400: '#a8846e',
            500: '#8b6f47',
            600: '#6b5435',
            700: '#4a3a24',
            800: '#2d2416',
            900: '#1a140c',
          },
          black: '#0a0908',
          gold: {
            300: '#ffd700',
            400: '#ffcc00',
            500: '#d4af37',
            600: '#b8941e',
          },
        },
        // Semantic color mappings
        primary: '#8b6f47',
        secondary: '#bfa490',
        accent: '#d4af37',
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        serif: ['Merriweather', 'Georgia', 'serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
        '128': '32rem',
      },
      boxShadow: {
        'sfs': '0 4px 6px rgba(139, 111, 71, 0.1)',
        'sfs-lg': '0 10px 15px rgba(139, 111, 71, 0.2)',
        'sfs-gold': '0 4px 6px rgba(212, 175, 55, 0.3)',
      },
      backgroundImage: {
        'sfs-gradient': 'linear-gradient(135deg, #8b6f47 0%, #d4af37 100%)',
        'sfs-gradient-dark': 'linear-gradient(135deg, #4a3a24 0%, #b8941e 100%)',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

## Component Style Examples

### Button Variants
```jsx
// Primary Button
<button className="bg-sfs-brown-500 hover:bg-sfs-gold-500 text-white font-semibold py-2 px-6 rounded-lg transition-colors duration-200 shadow-sfs">
  Primary Action
</button>

// Secondary Button
<button className="bg-sfs-brown-100 hover:bg-sfs-brown-200 text-sfs-brown-800 font-semibold py-2 px-6 rounded-lg transition-colors duration-200">
  Secondary Action
</button>

// Gold Accent Button
<button className="bg-sfs-gold-500 hover:bg-sfs-gold-600 text-sfs-black font-semibold py-2 px-6 rounded-lg transition-colors duration-200 shadow-sfs-gold">
  Special Action
</button>
```

### Card Components
```jsx
<div className="bg-white border border-sfs-brown-200 rounded-xl p-6 shadow-sfs hover:shadow-sfs-lg transition-shadow">
  <h3 className="text-2xl font-bold text-sfs-brown-800 mb-4">Card Title</h3>
  <p className="text-sfs-brown-600">Card content with proper text hierarchy.</p>
</div>
```

### Navigation
```jsx
<nav className="bg-sfs-black border-b border-sfs-gold-500">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div className="flex items-center justify-between h-16">
      <a href="/" className="text-sfs-gold-500 text-2xl font-bold">
        SmartFlow Systems
      </a>
      <div className="space-x-4">
        <a href="#" className="text-sfs-brown-50 hover:text-sfs-gold-400 transition-colors">
          Home
        </a>
        <a href="#" className="text-sfs-brown-50 hover:text-sfs-gold-400 transition-colors">
          Features
        </a>
      </div>
    </div>
  </div>
</nav>
```

### Form Inputs
```jsx
<input
  type="text"
  className="w-full px-4 py-2 border border-sfs-brown-200 rounded-lg focus:ring-2 focus:ring-sfs-gold-500 focus:border-transparent bg-white text-sfs-brown-800 placeholder-sfs-brown-300"
  placeholder="Enter text..."
/>
```

## CSS Global Styles

```css
/* src/styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    @apply bg-sfs-brown-50 text-sfs-brown-800;
  }

  h1, h2, h3, h4, h5, h6 {
    @apply text-sfs-brown-800 font-bold;
  }

  a {
    @apply text-sfs-brown-600 hover:text-sfs-gold-500 transition-colors;
  }
}

@layer components {
  .btn-primary {
    @apply bg-sfs-brown-500 hover:bg-sfs-gold-500 text-white font-semibold py-2 px-6 rounded-lg transition-colors duration-200 shadow-sfs;
  }

  .btn-secondary {
    @apply bg-sfs-brown-100 hover:bg-sfs-brown-200 text-sfs-brown-800 font-semibold py-2 px-6 rounded-lg transition-colors duration-200;
  }

  .btn-accent {
    @apply bg-sfs-gold-500 hover:bg-sfs-gold-600 text-sfs-black font-semibold py-2 px-6 rounded-lg transition-colors duration-200 shadow-sfs-gold;
  }

  .card {
    @apply bg-white border border-sfs-brown-200 rounded-xl p-6 shadow-sfs hover:shadow-sfs-lg transition-shadow;
  }

  .input {
    @apply w-full px-4 py-2 border border-sfs-brown-200 rounded-lg focus:ring-2 focus:ring-sfs-gold-500 focus:border-transparent bg-white text-sfs-brown-800 placeholder-sfs-brown-300;
  }
}
```

## Execution Steps

1. **Check Current Setup**
   - Scan for existing Tailwind configuration
   - Identify CSS framework in use
   - Check for conflicting color systems

2. **Install Dependencies**
   ```bash
   npm install -D tailwindcss postcss autoprefixer
   npm install -D @tailwindcss/forms @tailwindcss/typography
   npx tailwindcss init -p
   ```

3. **Apply Configuration**
   - Update `tailwind.config.js` with SFS colors
   - Configure content paths
   - Add theme extensions

4. **Create Global Styles**
   - Set up `globals.css` with SFS base styles
   - Define component classes
   - Add utility extensions

5. **Update Components**
   - Scan existing components
   - Replace color classes with SFS palette
   - Update hover/focus states

6. **Add Brand Assets**
   - Reference logo files
   - Set up favicon
   - Configure meta tags for social sharing

7. **Test Theme**
   - Verify color contrast (WCAG AA)
   - Test interactive states
   - Check responsive behavior

8. **Generate Theme Documentation**
   - Create component examples
   - Document color usage guidelines
   - Add accessibility notes

## Brand Asset Locations

```
public/
├── favicon.ico
├── logo.svg           # Main SFS logo
├── logo-white.svg     # White variant for dark backgrounds
├── logo-icon.svg      # Icon only
└── og-image.png       # Social media preview (1200x630)
```

## Accessibility Standards

All SFS theme implementations must meet:
- WCAG 2.1 Level AA compliance
- Minimum 4.5:1 contrast ratio for normal text
- Minimum 3:1 contrast ratio for large text
- Focus indicators visible on all interactive elements
- Color not used as the only means of conveying information

## Color Contrast Validation

```javascript
// Safe color combinations for accessibility
const accessiblePairs = [
  { bg: 'sfs-brown-50', text: 'sfs-brown-800' },    // ✓ 12.8:1
  { bg: 'sfs-brown-500', text: 'white' },            // ✓ 4.7:1
  { bg: 'sfs-black', text: 'sfs-gold-300' },         // ✓ 9.2:1
  { bg: 'sfs-gold-500', text: 'sfs-black' },         // ✓ 8.1:1
  { bg: 'white', text: 'sfs-brown-600' },            // ✓ 6.3:1
]
```

## Dark Mode Configuration (Optional)

```javascript
// Add to tailwind.config.js
module.exports = {
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        // Dark mode specific colors
        'dark-bg': '#1a140c',
        'dark-surface': '#2d2416',
        'dark-text': '#e8e2dc',
      },
    },
  },
}
```

```css
/* Dark mode styles */
@media (prefers-color-scheme: dark) {
  .dark body {
    @apply bg-sfs-brown-900 text-sfs-brown-100;
  }
}
```

## Completion Checklist

After applying theme:
- [ ] Tailwind CSS installed and configured
- [ ] SFS color palette added to config
- [ ] Global styles created
- [ ] Component classes defined
- [ ] Interactive states styled (hover, focus, active)
- [ ] Accessibility validated (WCAG AA)
- [ ] Brand assets referenced
- [ ] Documentation updated
- [ ] Theme tested across browsers
- [ ] Responsive behavior verified

## Common Theme Issues & Fixes

### Issue: Colors not applying
**Fix:** Ensure Tailwind content paths include all template files
```javascript
content: ['./src/**/*.{js,jsx,ts,tsx}', './public/index.html']
```

### Issue: Hover states not smooth
**Fix:** Add transition utilities
```jsx
className="transition-colors duration-200"
```

### Issue: Low contrast warnings
**Fix:** Use validated accessible color pairs from the table above

## Related SFS Skills
- `sfs-repo-setup` - Full repository initialization
- `sfs-readme-gen` - Generate branded documentation
- `sfs-design-system` - Create component library

## References
- SFS Brand Assets: `/home/garet/SFS/sfs-brand-assets/`
- Tailwind Docs: https://tailwindcss.com/docs
- WCAG Guidelines: https://www.w3.org/WAI/WCAG21/quickref/
