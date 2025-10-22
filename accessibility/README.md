# Web Accessibility (a11y)

_Building for everyone, not just some._

## Overview

Accessibility isn't a feature you add at the end. It's a fundamental part of building software that puts people first. When we build inaccessible applications, we're not just creating a bad experience — we're actively excluding people from participating in digital life.

This isn't about compliance checklists or legal requirements (though those matter). It's about recognizing that **disability is part of the human experience**, and our job as engineers is to build systems that work for everyone.

---

## The Building Metaphor

**Inaccessible web app:**
You've built a beautiful building with marble floors, floor-to-ceiling windows, and stunning architecture. But it only has stairs. No ramps, no elevators, no clear signage. People using wheelchairs can't enter. People with visual impairments can't navigate. People carrying heavy boxes struggle. You've built for an imaginary "average person" who doesn't exist.

**Accessible web app:**
Same beautiful building, but with:
- **Ramps and elevators** (keyboard navigation, screen reader support)
- **Clear signage** (semantic HTML, ARIA labels)
- **Multiple ways to navigate** (skip links, landmarks, headings)
- **Adjustable features** (text sizing, contrast modes)
- **Sensory alternatives** (captions for video, alt text for images)

Everyone can use the building. And often, the accessible features help everyone — parents with strollers, people with temporary injuries, people in bright sunlight who need contrast.

**Universal design benefits everyone.**

---

## Core Principles (POUR)

The Web Content Accessibility Guidelines (WCAG) are built on four principles. Every interface should be:

### 1. **Perceivable**
Information must be presentable to users in ways they can perceive.

**Not just:** "Can they see it?"
**But:** "Can they perceive it through sight, sound, or touch?"

**Examples:**
- ✅ Images have alt text for screen readers
- ✅ Videos have captions for deaf users
- ✅ Color isn't the only way to convey information
- ❌ Red/green for success/error with no text label

### 2. **Operable**
Users must be able to operate the interface.

**Not just:** "Can they click it?"
**But:** "Can they interact with it using mouse, keyboard, voice, or assistive tech?"

**Examples:**
- ✅ All functionality available via keyboard
- ✅ Users have enough time to read and interact
- ✅ Nothing causes seizures (no rapid flashing)
- ❌ Dropdown only works with mouse hover

### 3. **Understandable**
Information and interface operation must be understandable.

**Not just:** "Is the code clear?"
**But:** "Can users understand what things do and how to use them?"

**Examples:**
- ✅ Error messages explain what went wrong and how to fix it
- ✅ Labels clearly describe what inputs need
- ✅ Navigation is predictable and consistent
- ❌ Button labeled "Submit" that actually deletes data

### 4. **Robust**
Content must work across assistive technologies.

**Not just:** "Does it work in Chrome?"
**But:** "Does it work with screen readers, voice control, and future tech?"

**Examples:**
- ✅ Uses semantic HTML that assistive tech understands
- ✅ ARIA used correctly (not overdone)
- ✅ Works across browsers and assistive tech
- ❌ Custom `<div onclick>` buttons that screen readers can't interact with

---

## Practical Patterns

### Pattern 1: Semantic HTML First

**The Problem:**
Divs and spans everywhere. Screen readers can't distinguish a button from a heading from a navigation menu.

**The Solution:**
Use HTML elements for their intended purpose.

```html
<!-- ❌ Bad: Div soup -->
<div class="header">
  <div class="nav">
    <div class="nav-item" onclick="navigate()">Home</div>
    <div class="nav-item" onclick="navigate()">About</div>
  </div>
</div>
<div class="main">
  <div class="heading">Welcome</div>
  <div class="button" onclick="submit()">Submit</div>
</div>

<!-- ✅ Good: Semantic HTML -->
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
<main>
  <h1>Welcome</h1>
  <button type="submit">Submit</button>
</main>
```

**Why This Matters:**
- Screen readers announce element types ("heading level 1", "button", "navigation")
- Keyboard navigation works automatically (buttons/links are focusable)
- Browser features work (right-click to open link in new tab)
- CSS cascade makes more sense

**Semantic Elements to Use:**
- `<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>` - Page structure
- `<h1>` through `<h6>` - Heading hierarchy
- `<button>` - Actions
- `<a>` - Navigation
- `<article>`, `<section>` - Content grouping
- `<form>`, `<label>`, `<input>` - Forms
- `<table>`, `<thead>`, `<tbody>` - Tabular data

---

### Pattern 2: Keyboard Navigation

**The Problem:**
Many users can't use a mouse. They navigate with keyboard, voice control, or other assistive tech.

**The Solution:**
Ensure all functionality is keyboard accessible.

```typescript
// ❌ Bad: Hover-only interactions
<div
  onMouseEnter={() => setShowMenu(true)}
  onMouseLeave={() => setShowMenu(false)}
>
  Hover for menu
</div>

// ✅ Good: Keyboard accessible
<button
  onClick={() => setShowMenu(!showMenu)}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      setShowMenu(!showMenu);
    }
    if (e.key === 'Escape') {
      setShowMenu(false);
    }
  }}
  aria-expanded={showMenu}
  aria-haspopup="true"
>
  Toggle menu
</button>
```

**Keyboard Interactions to Support:**

| Element | Keys | Behavior |
|---------|------|----------|
| **Button** | Enter, Space | Activate |
| **Link** | Enter | Navigate |
| **Checkbox** | Space | Toggle |
| **Radio** | Arrow keys | Select option |
| **Dropdown** | Arrow keys, Enter, Esc | Navigate options |
| **Modal** | Esc | Close |
| **Tabs** | Arrow keys | Switch tabs |

**Testing Your Keyboard Nav:**
```
1. Unplug your mouse
2. Use only Tab, Shift+Tab, Enter, Space, Arrow keys, Esc
3. Can you:
   - Navigate to every interactive element?
   - See where focus is at all times?
   - Activate all buttons and links?
   - Fill out and submit forms?
   - Close modals and dropdowns?
   - Use your entire application?

If not, you have keyboard accessibility issues.
```

**Focus Management:**

```typescript
// ❌ Bad: No visible focus indicator
button {
  outline: none; /* DON'T DO THIS */
}

// ✅ Good: Clear focus indicator
button:focus-visible {
  outline: 3px solid #0066cc;
  outline-offset: 2px;
}

// ✅ Even better: Custom but still clear
button:focus-visible {
  box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.5);
  outline: none; /* Okay to remove if you have a custom indicator */
}
```

**Focus Trap for Modals:**

```typescript
function Modal({ isOpen, onClose, children }) {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isOpen) return;

    const modal = modalRef.current;
    if (!modal) return;

    // Get all focusable elements in modal
    const focusableElements = modal.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const firstElement = focusableElements[0] as HTMLElement;
    const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

    // Focus first element when modal opens
    firstElement?.focus();

    // Trap focus inside modal
    const handleTab = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;

      if (e.shiftKey) {
        // Shift + Tab
        if (document.activeElement === firstElement) {
          e.preventDefault();
          lastElement?.focus();
        }
      } else {
        // Tab
        if (document.activeElement === lastElement) {
          e.preventDefault();
          firstElement?.focus();
        }
      }
    };

    // Close on Escape
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };

    modal.addEventListener('keydown', handleTab);
    modal.addEventListener('keydown', handleEscape);

    return () => {
      modal.removeEventListener('keydown', handleTab);
      modal.removeEventListener('keydown', handleEscape);
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      <h2 id="modal-title">Modal Title</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

---

### Pattern 3: Screen Reader Support

**The Problem:**
Visual design conveys meaning that screen readers can't see (icons, colors, layout).

**The Solution:**
Provide text alternatives and semantic cues.

**Alt Text for Images:**

```html
<!-- ❌ Bad: No alt text -->
<img src="chart.png">

<!-- ❌ Bad: Redundant alt text -->
<img src="chart.png" alt="Image of a chart">

<!-- ✅ Good: Descriptive alt text -->
<img src="chart.png" alt="Bar chart showing 40% increase in sales from Q1 to Q2 2024">

<!-- ✅ Good: Decorative images -->
<img src="decorative-line.png" alt="" role="presentation">
```

**Alt Text Guidelines:**
- **Informative images:** Describe the content/function
- **Functional images (icons):** Describe the action ("Close", not "X icon")
- **Decorative images:** Use empty alt (`alt=""`) so screen readers skip them
- **Complex images (charts, diagrams):** Use alt for summary, provide detailed description nearby

**ARIA Labels:**

```html
<!-- Icon buttons need labels -->
<button aria-label="Close dialog">
  <CloseIcon />
</button>

<!-- Search input with icon -->
<label>
  <span class="visually-hidden">Search</span>
  <input type="search" placeholder="Search..." />
</label>

<!-- Or use aria-label -->
<input
  type="search"
  aria-label="Search products"
  placeholder="Search..."
/>

<!-- Landmark regions -->
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer navigation">...</nav>

<!-- Form fields -->
<label for="email">Email address</label>
<input
  id="email"
  type="email"
  aria-describedby="email-help"
  aria-required="true"
/>
<span id="email-help">We'll never share your email</span>
```

**Visually Hidden Text:**

```css
/* Screen reader only text */
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

```html
<!-- Status messages for screen readers -->
<button>
  Delete
  <span class="visually-hidden">user account</span>
</button>
<!-- Announces: "Delete user account" -->

<!-- Loading states -->
<div aria-live="polite" aria-atomic="true">
  <span class="visually-hidden">
    {isLoading ? 'Loading content...' : 'Content loaded'}
  </span>
</div>
```

---

### Pattern 4: ARIA (When HTML Isn't Enough)

**The Golden Rule:** No ARIA is better than bad ARIA.

**When to Use ARIA:**
1. HTML doesn't have a semantic element for what you need (tabs, accordions)
2. You need to communicate dynamic changes (live regions)
3. You need to add context HTML can't provide (labels, descriptions)

**When NOT to Use ARIA:**
- ❌ `<div role="button">` when `<button>` exists
- ❌ `<span role="heading">` when `<h1-h6>` exist
- ❌ `aria-label` on elements that already have visible text

**Common ARIA Patterns:**

**Accordion:**
```html
<div class="accordion">
  <h3>
    <button
      id="accordion1-trigger"
      aria-expanded="false"
      aria-controls="accordion1-content"
    >
      Section 1
    </button>
  </h3>
  <div
    id="accordion1-content"
    role="region"
    aria-labelledby="accordion1-trigger"
    hidden
  >
    Content for section 1
  </div>
</div>
```

**Tabs:**
```html
<div class="tabs">
  <div role="tablist" aria-label="Product information">
    <button
      role="tab"
      aria-selected="true"
      aria-controls="description-panel"
      id="description-tab"
    >
      Description
    </button>
    <button
      role="tab"
      aria-selected="false"
      aria-controls="reviews-panel"
      id="reviews-tab"
      tabindex="-1"
    >
      Reviews
    </button>
  </div>

  <div
    id="description-panel"
    role="tabpanel"
    aria-labelledby="description-tab"
  >
    Product description...
  </div>

  <div
    id="reviews-panel"
    role="tabpanel"
    aria-labelledby="reviews-tab"
    hidden
  >
    Customer reviews...
  </div>
</div>
```

**Live Regions (Dynamic Content):**
```html
<!-- Polite: Announce when user is idle -->
<div aria-live="polite">
  {statusMessage}
</div>

<!-- Assertive: Announce immediately (use sparingly!) -->
<div aria-live="assertive" role="alert">
  {errorMessage}
</div>

<!-- Example: Form errors -->
<div role="alert" aria-live="assertive">
  {errors.map(error => <p key={error}>{error}</p>)}
</div>
```

**ARIA States:**
- `aria-expanded` - Whether element is expanded (dropdowns, accordions)
- `aria-selected` - Currently selected item (tabs, listbox)
- `aria-checked` - Checkbox/radio state
- `aria-pressed` - Toggle button state
- `aria-current` - Current item in navigation
- `aria-disabled` - Disabled but still visible
- `aria-hidden` - Hidden from assistive tech (use carefully!)

---

### Pattern 5: Color Contrast & Visual Design

**The Problem:**
Low contrast text is hard or impossible to read for people with low vision or color blindness.

**The Solution:**
Meet WCAG contrast requirements.

**Contrast Ratios:**

| Text Size | WCAG AA | WCAG AAA |
|-----------|---------|----------|
| **Normal text** (< 18px or < 14px bold) | 4.5:1 | 7:1 |
| **Large text** (≥ 18px or ≥ 14px bold) | 3:1 | 4.5:1 |
| **UI components** (icons, borders) | 3:1 | - |

```css
/* ❌ Bad: Insufficient contrast */
.text {
  color: #999999; /* 2.8:1 against white */
  background: #ffffff;
}

/* ✅ Good: Meets WCAG AA */
.text {
  color: #757575; /* 4.5:1 against white */
  background: #ffffff;
}

/* ✅ Better: Meets WCAG AAA */
.text {
  color: #595959; /* 7:1 against white */
  background: #ffffff;
}
```

**Don't Rely on Color Alone:**

```html
<!-- ❌ Bad: Color only -->
<p style="color: red;">Error: Invalid input</p>
<p style="color: green;">Success: Saved</p>

<!-- ✅ Good: Color + icon + text -->
<p class="error">
  <ErrorIcon aria-hidden="true" />
  <strong>Error:</strong> Invalid input
</p>
<p class="success">
  <SuccessIcon aria-hidden="true" />
  <strong>Success:</strong> Saved
</p>
```

**Form Validation:**

```tsx
function FormField({ error }: { error?: string }) {
  return (
    <div>
      <label htmlFor="email">
        Email
        {error && <span aria-label="Error"> *</span>}
      </label>
      <input
        id="email"
        type="email"
        aria-invalid={!!error}
        aria-describedby={error ? "email-error" : undefined}
        className={error ? 'input-error' : ''}
      />
      {error && (
        <span id="email-error" role="alert" className="error-message">
          <ErrorIcon aria-hidden="true" />
          {error}
        </span>
      )}
    </div>
  );
}
```

**Testing Contrast:**
- Chrome DevTools: Inspect element → Check contrast ratio
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Colorblind simulator](https://www.toptal.com/designers/colorfilter)

---

### Pattern 6: Forms & Error Handling

**The Problem:**
Forms are often the most inaccessible part of web apps.

**The Solution:**
Clear labels, helpful errors, logical structure.

**Accessible Form Example:**

```html
<form>
  <!-- Always associate labels with inputs -->
  <div class="form-field">
    <label for="name">
      Full Name
      <span aria-label="required">*</span>
    </label>
    <input
      id="name"
      type="text"
      required
      aria-required="true"
      aria-describedby="name-hint"
    />
    <span id="name-hint" class="hint">First and last name</span>
  </div>

  <!-- Grouping related inputs -->
  <fieldset>
    <legend>Contact preference</legend>
    <div>
      <input
        type="radio"
        id="contact-email"
        name="contact"
        value="email"
      />
      <label for="contact-email">Email</label>
    </div>
    <div>
      <input
        type="radio"
        id="contact-phone"
        name="contact"
        value="phone"
      />
      <label for="contact-phone">Phone</label>
    </div>
  </fieldset>

  <!-- Error handling -->
  <div class="form-field">
    <label for="email">Email</label>
    <input
      id="email"
      type="email"
      aria-invalid="true"
      aria-describedby="email-error"
    />
    <span id="email-error" role="alert" class="error">
      Please enter a valid email address
    </span>
  </div>

  <button type="submit">Submit</button>
</form>
```

**Error Summary (for complex forms):**

```html
<!-- Show all errors at top of form -->
<div role="alert" aria-labelledby="error-heading">
  <h2 id="error-heading">There are 2 errors in your form:</h2>
  <ul>
    <li><a href="#email">Email: Please enter a valid email</a></li>
    <li><a href="#password">Password: Must be at least 8 characters</a></li>
  </ul>
</div>
```

---

## Testing for Accessibility

### Automated Testing

**Tools:**
- **[axe DevTools](https://www.deque.com/axe/devtools/)** - Browser extension
- **[Lighthouse](https://developers.google.com/web/tools/lighthouse)** - Built into Chrome DevTools
- **[WAVE](https://wave.webaim.org/)** - Browser extension
- **[@axe-core/react](https://github.com/dequelabs/axe-core-npm)** - Runtime checking in development

```typescript
// Install axe-core for automated testing
import { configureAxe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('should have no accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

**Remember:** Automated tools only catch ~30-40% of accessibility issues.

### Manual Testing

**1. Keyboard Navigation Test:**
- Unplug your mouse
- Tab through entire application
- Check all interactive elements are reachable and usable

**2. Screen Reader Test:**
- **Mac:** VoiceOver (Cmd + F5)
- **Windows:** NVDA (free) or JAWS
- **Chrome/Edge:** ChromeVox extension

**3. Zoom Test:**
- Zoom browser to 200%
- Check layout doesn't break
- Check text is still readable
- Check no content is hidden

**4. Color Blindness Test:**
- Use a color blindness simulator
- Ensure information isn't color-dependent

**5. Reduce Motion:**
```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Common Accessibility Mistakes

### 1. Div/Span Buttons
```html
<!-- ❌ Screen readers can't interact -->
<div onClick={handleClick}>Click me</div>

<!-- ✅ Use actual buttons -->
<button onClick={handleClick}>Click me</button>
```

### 2. Missing Alt Text
```html
<!-- ❌ No context for screen readers -->
<img src="icon.png">

<!-- ✅ Descriptive alt text -->
<img src="icon.png" alt="Settings icon">
```

### 3. Poor Focus Indicators
```css
/* ❌ Removes focus outline -->
* { outline: none; }

/* ✅ Custom but visible -->
button:focus-visible {
  outline: 2px solid #0066cc;
}
```

### 4. Non-Descriptive Links
```html
<!-- ❌ "Click here" is meaningless out of context -->
<a href="/docs">Click here</a> for documentation

<!-- ✅ Link text describes destination -->
<a href="/docs">Read the documentation</a>
```

### 5. Auto-playing Media
```html
<!-- ❌ Can't be stopped -->
<video autoplay src="video.mp4"></video>

<!-- ✅ User controls -->
<video controls src="video.mp4"></video>
```

### 6. Time Limits Without Warning
```typescript
// ❌ Session expires without warning
setTimeout(logout, 300000);

// ✅ Warn user, allow extension
function handleSessionTimeout() {
  showModal({
    title: 'Session expiring soon',
    message: 'Your session will expire in 1 minute.',
    actions: ['Extend session', 'Log out']
  });
}
```

---

## Connection to Responsible AI & Equity

Accessibility is equity in practice. It's the [Responsible AI framework](../responsible-ai/README.md) applied to user interfaces:

### **Equity Must Be a Feature, Not a Footnote**
Building accessible applications ensures everyone can participate, regardless of ability. Inaccessible software perpetuates inequality.

### **Human Dignity Is Not Negotiable**
When we build inaccessible applications, we're saying some users don't matter. That's unacceptable.

### **Technology Should Expand Opportunity, Not Limit It**
Accessibility opens doors. Inaccessibility locks people out of jobs, education, services, and community.

---

## Resources

### Guidelines & Standards
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/) - The standard
- [A11y Project Checklist](https://www.a11yproject.com/checklist/) - Practical checklist
- [MDN Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility) - Developer docs

### Testing Tools
- [axe DevTools](https://www.deque.com/axe/devtools/) - Automated testing
- [WAVE](https://wave.webaim.org/) - Visual feedback tool
- [NVDA](https://www.nvaccess.org/) - Free screen reader (Windows)
- [Contrast Checker](https://webaim.org/resources/contrastchecker/) - Color contrast

### Learning
- [WebAIM](https://webaim.org/) - Articles and training
- [Inclusive Components](https://inclusive-components.design/) - Accessible patterns
- [A11y Coffee](https://a11y.coffee/) - Quick accessibility tips
- [Deque University](https://dequeuniversity.com/) - In-depth courses

### Component Libraries
- [Radix UI](https://www.radix-ui.com/) - Unstyled, accessible components
- [Headless UI](https://headlessui.com/) - Accessible React/Vue components
- [Reach UI](https://reach.tech/) - Accessible React components
- [Chakra UI](https://chakra-ui.com/) - Accessible component library

---

## The Bottom Line

Accessibility isn't:
- ❌ A nice-to-have feature
- ❌ Something to add at the end
- ❌ Just for people with disabilities
- ❌ Hard or expensive

Accessibility is:
- ✅ A fundamental requirement for inclusive software
- ✅ Built in from the start
- ✅ Better UX for everyone (parents with strollers benefit from ramps too)
- ✅ Often simpler (semantic HTML is easier than div soup)

**Universal design benefits everyone.** Curb cuts help wheelchair users, but also people with strollers, luggage, bikes, and delivery carts. Captions help deaf users, but also people in noisy environments or learning a new language.

When we build accessible software, we build better software.

> **"The power of the Web is in its universality. Access by everyone regardless of disability is an essential aspect."** — Tim Berners-Lee, inventor of the World Wide Web

---

_Building for everyone: Where empathy meets engineering._
