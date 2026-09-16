# Component Patterns

Reusable UI component implementation patterns for prototypes. All components use vanilla HTML/CSS/JS with CSS custom properties for theming.

## Navigation Components

### Top Navigation Bar
```html
<header class="navbar">
  <div class="navbar-container">
    <div class="navbar-brand">
      <svg class="brand-logo" width="32" height="32"><!-- logo SVG --></svg>
      <span class="brand-name">AppName</span>
    </div>
    <nav class="navbar-menu">
      <a href="#" class="nav-link active">Dashboard</a>
      <a href="#" class="nav-link">Projects</a>
      <a href="#" class="nav-link">Team</a>
      <a href="#" class="nav-link">Settings</a>
    </nav>
    <div class="navbar-actions">
      <button class="btn-icon" aria-label="Notifications">
        <svg><!-- bell icon --></svg>
        <span class="badge-dot"></span>
      </button>
      <div class="avatar" style="background: var(--color-primary)">JD</div>
    </div>
  </div>
</header>
```

### Sidebar Navigation (Collapsible)
```html
<aside class="sidebar" id="sidebar">
  <div class="sidebar-header">
    <span class="sidebar-title">Menu</span>
    <button class="sidebar-toggle" onclick="toggleSidebar()">
      <svg><!-- hamburger icon --></svg>
    </button>
  </div>
  <nav class="sidebar-nav">
    <div class="nav-group">
      <span class="nav-group-label">Main</span>
      <a href="#" class="sidebar-link active">
        <svg class="sidebar-icon"><!-- icon --></svg>
        <span class="sidebar-text">Dashboard</span>
      </a>
      <!-- more links -->
    </div>
  </nav>
</aside>
```

### Mobile Bottom Tab Bar
```html
<nav class="bottom-tabs">
  <a href="#home" class="tab-item active">
    <svg class="tab-icon"><!-- home icon --></svg>
    <span class="tab-label">Home</span>
  </a>
  <a href="#search" class="tab-item">
    <svg class="tab-icon"><!-- search icon --></svg>
    <span class="tab-label">Search</span>
  </a>
  <button class="tab-item tab-fab" aria-label="Create">
    <svg><!-- plus icon --></svg>
  </button>
  <a href="#activity" class="tab-item">
    <svg class="tab-icon"><!-- activity icon --></svg>
    <span class="tab-label">Activity</span>
  </a>
  <a href="#profile" class="tab-item">
    <svg class="tab-icon"><!-- user icon --></svg>
    <span class="tab-label">Profile</span>
  </a>
</nav>
```

## Card Components

### Stat Card (Dashboard KPI)
```html
<div class="stat-card">
  <div class="stat-header">
    <span class="stat-label">Total Revenue</span>
    <div class="stat-icon" style="background: var(--color-success-bg)">
      <svg style="color: var(--color-success)"><!-- dollar icon --></svg>
    </div>
  </div>
  <div class="stat-value">¥284,529</div>
  <div class="stat-trend trend-up">
    <svg><!-- arrow up --></svg>
    <span>+12.5%</span>
    <span class="trend-label">vs last month</span>
  </div>
</div>
```

### Content Card
```html
<div class="card">
  <div class="card-header">
    <h3 class="card-title">Recent Orders</h3>
    <button class="btn-text">View all</button>
  </div>
  <div class="card-body">
    <div class="list-item">
      <div class="avatar">A</div>
      <div class="list-content">
        <div class="list-title">Order #1024</div>
        <div class="list-subtitle">Alice Chen · 2 items</div>
      </div>
      <div class="list-meta">
        <span class="badge badge-success">Completed</span>
        <span class="list-amount">¥299.00</span>
      </div>
    </div>
    <!-- more items -->
  </div>
</div>
```

### Feature Card (Landing Page)
```html
<div class="feature-card">
  <div class="feature-icon" style="background: var(--color-primary-light)">
    <svg style="color: var(--color-primary)" width="24" height="24"><!-- icon --></svg>
  </div>
  <h3 class="feature-title">Lightning Fast</h3>
  <p class="feature-description">
    Process millions of records in seconds with our optimized engine.
  </p>
  <a href="#" class="feature-link">Learn more →</a>
</div>
```

## Form Components

### Text Input
```html
<div class="form-group">
  <label class="form-label" for="email">Email Address</label>
  <div class="input-wrapper">
    <svg class="input-icon"><!-- mail icon --></svg>
    <input
      type="email"
      id="email"
      class="form-input"
      placeholder="you@example.com"
      required
    >
  </div>
  <span class="form-hint">We'll never share your email.</span>
</div>
```

### Select Dropdown
```html
<div class="form-group">
  <label class="form-label" for="role">Role</label>
  <select id="role" class="form-select">
    <option value="">Select a role...</option>
    <option value="admin">Administrator</option>
    <option value="editor">Editor</option>
    <option value="viewer">Viewer</option>
  </select>
</div>
```

### Toggle Switch
```html
<div class="form-row">
  <div class="form-info">
    <span class="form-label">Email Notifications</span>
    <span class="form-hint">Receive updates about your account</span>
  </div>
  <label class="toggle">
    <input type="checkbox" checked>
    <span class="toggle-slider"></span>
  </label>
</div>
```

### Search Bar
```html
<div class="search-bar">
  <svg class="search-icon"><!-- search icon --></svg>
  <input type="text" class="search-input" placeholder="Search...">
  <kbd class="search-shortcut">⌘K</kbd>
</div>
```

## Data Display

### Data Table
```html
<div class="table-container">
  <table class="data-table">
    <thead>
      <tr>
        <th class="th-checkbox"><input type="checkbox"></th>
        <th class="sortable">Order ID <svg><!-- sort icon --></svg></th>
        <th class="sortable">Customer</th>
        <th>Product</th>
        <th class="sortable">Date</th>
        <th>Status</th>
        <th>Amount</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><input type="checkbox"></td>
        <td class="cell-mono">#ORD-1024</td>
        <td>
          <div class="cell-user">
            <div class="avatar avatar-sm">AC</div>
            <span>Alice Chen</span>
          </div>
        </td>
        <td>Pro Plan (Annual)</td>
        <td>2026-07-15</td>
        <td><span class="badge badge-success">Completed</span></td>
        <td class="cell-amount">¥1,299.00</td>
        <td>
          <button class="btn-icon btn-sm" aria-label="More">
            <svg><!-- dots icon --></svg>
          </button>
        </td>
      </tr>
    </tbody>
  </table>
  <div class="table-pagination">
    <span class="pagination-info">Showing 1-10 of 248</span>
    <div class="pagination-controls">
      <button class="btn-icon" disabled>←</button>
      <button class="page-btn active">1</button>
      <button class="page-btn">2</button>
      <button class="page-btn">3</button>
      <button class="btn-icon">→</button>
    </div>
  </div>
</div>
```

### Bar Chart (Pure CSS)
```html
<div class="chart-container">
  <div class="chart-bars">
    <div class="chart-bar-group">
      <div class="chart-bar" style="height: 60%" title="Mon: 240">
        <span class="chart-bar-value">240</span>
      </div>
      <span class="chart-bar-label">Mon</span>
    </div>
    <div class="chart-bar-group">
      <div class="chart-bar" style="height: 85%" title="Tue: 340">
        <span class="chart-bar-value">340</span>
      </div>
      <span class="chart-bar-label">Tue</span>
    </div>
    <!-- more bars -->
  </div>
</div>
```

### Donut Chart (SVG)
```html
<div class="donut-chart">
  <svg width="160" height="160" viewBox="0 0 160 160">
    <circle cx="80" cy="80" r="60" fill="none" stroke="var(--color-surface)" stroke-width="20"/>
    <circle cx="80" cy="80" r="60" fill="none"
      stroke="var(--color-primary)" stroke-width="20"
      stroke-dasharray="226 377"
      stroke-dashoffset="0"
      transform="rotate(-90 80 80)"/>
    <circle cx="80" cy="80" r="60" fill="none"
      stroke="var(--color-secondary)" stroke-width="20"
      stroke-dasharray="113 377"
      stroke-dashoffset="-226"
      transform="rotate(-90 80 80)"/>
  </svg>
  <div class="donut-center">
    <span class="donut-value">1,284</span>
    <span class="donut-label">Total</span>
  </div>
</div>
```

### Progress Bar
```html
<div class="progress">
  <div class="progress-header">
    <span class="progress-label">Storage Used</span>
    <span class="progress-value">7.2 / 10 GB</span>
  </div>
  <div class="progress-track">
    <div class="progress-fill" style="width: 72%"></div>
  </div>
</div>
```

## Feedback Components

### Modal Dialog
```html
<div class="modal-overlay" id="modal" style="display:none">
  <div class="modal">
    <div class="modal-header">
      <h2 class="modal-title">Create New Project</h2>
      <button class="btn-icon" onclick="closeModal()" aria-label="Close">
        <svg><!-- x icon --></svg>
      </button>
    </div>
    <div class="modal-body">
      <!-- form content -->
    </div>
    <div class="modal-footer">
      <button class="btn btn-secondary" onclick="closeModal()">Cancel</button>
      <button class="btn btn-primary">Create Project</button>
    </div>
  </div>
</div>
```

### Toast Notification
```html
<div class="toast-container" id="toastContainer">
  <!-- Toasts injected here -->
</div>

<script>
function showToast(message, type = 'info') {
  const toast = document.createElement('div');
  toast.className = `toast toast-${type}`;
  toast.innerHTML = `
    <svg class="toast-icon"><!-- icon --></svg>
    <span class="toast-message">${message}</span>
    <button class="toast-close" aria-label="Close">×</button>
  `;
  document.getElementById('toastContainer').appendChild(toast);
  setTimeout(() => toast.classList.add('toast-show'), 10);
  setTimeout(() => {
    toast.classList.remove('toast-show');
    setTimeout(() => toast.remove(), 300);
  }, 3000);
}
</script>
```

### Badge
```html
<span class="badge badge-success">Active</span>
<span class="badge badge-warning">Pending</span>
<span class="badge badge-error">Failed</span>
<span class="badge badge-info">New</span>
<span class="badge badge-neutral">Draft</span>
```

### Empty State
```html
<div class="empty-state">
  <div class="empty-icon">
    <svg width="64" height="64"><!-- illustration --></svg>
  </div>
  <h3 class="empty-title">No projects yet</h3>
  <p class="empty-description">
    Get started by creating your first project.
  </p>
  <button class="btn btn-primary">
    <svg><!-- plus icon --></svg>
    Create Project
  </button>
</div>
```

## Button Variants
```html
<!-- Primary -->
<button class="btn btn-primary">Save Changes</button>

<!-- Secondary -->
<button class="btn btn-secondary">Cancel</button>

<!-- Ghost / Text -->
<button class="btn btn-ghost">View Details</button>

<!-- Danger -->
<button class="btn btn-danger">Delete</button>

<!-- With Icon -->
<button class="btn btn-primary">
  <svg><!-- plus icon --></svg>
  New Project
</button>

<!-- Icon Only -->
<button class="btn-icon" aria-label="Settings">
  <svg><!-- gear icon --></svg>
</button>

<!-- Sizes -->
<button class="btn btn-primary btn-sm">Small</button>
<button class="btn btn-primary">Default</button>
<button class="btn btn-primary btn-lg">Large</button>
```

## Accordion
```html
<div class="accordion">
  <div class="accordion-item">
    <button class="accordion-header" onclick="toggleAccordion(this)">
      <span>What is included in the Pro plan?</span>
      <svg class="accordion-chevron"><!-- chevron --></svg>
    </button>
    <div class="accordion-content">
      <p>The Pro plan includes unlimited projects, advanced analytics, priority support, and team collaboration features for up to 10 members.</p>
    </div>
  </div>
  <!-- more items -->
</div>
```
