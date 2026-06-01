## Critical Button & Function Fixes

### Issue 1: Missing Core Helper Functions

**Location**: Before the `populateAllSelects()` call (around line 1810)

**Add this code block**:

```javascript
    // ===== MISSING HELPER FUNCTIONS =====
    
    function escapeHtml(text) {
      const div = document.createElement('div');
      div.textContent = text || '';
      return div.innerHTML;
    }

    function persistLessons() {
      localStorage.setItem('lessonsData', JSON.stringify(lessons));
      localStorage.setItem('optionsData', JSON.stringify(options));
    }

    function loadLessons() {
      const stored = localStorage.getItem('lessonsData');
      const storedOpts = localStorage.getItem('optionsData');
      if (stored) {
        lessons = JSON.parse(stored);
      }
      if (storedOpts) {
        options = JSON.parse(storedOpts);
      }
    }

    function getSelectedValue(containerId) {
      const container = document.getElementById(containerId);
      if (!container) return '';
      const btn = container.querySelector('.select-button span:first-child');
      return btn ? btn.textContent.trim() : '';
    }

    function selectedValues(containerId) {
      const container = document.getElementById(containerId);
      if (!container) return [];
      
      // Try to find selected options (for custom selects)
      const opts = container.querySelectorAll('.select-option.selected');
      if (opts.length > 0) {
        return Array.from(opts).map(o => o.textContent.trim());
      }
      
      // Fallback: check button text
      const btn = container.querySelector('.select-button span:first-child');
      if (btn && btn.textContent.trim() !== 'Select...') {
        return btn.textContent.split(',').map(v => v.trim()).filter(v => v);
      }
      return [];
    }

    function openEntryModal() {
      const modal = document.getElementById('entryModal');
      if (modal) {
        modal.style.display = 'flex';
        document.body.style.overflow = 'hidden';
      }
    }

    function closeEntryModal() {
      const modal = document.getElementById('entryModal');
      if (modal) {
        modal.style.display = 'none';
        document.body.style.overflow = 'auto';
      }
    }

    function closeModalOnBackdropClick(event) {
      if (event.target === event.currentTarget) {
        if (event.currentTarget.id === 'entryModalBackdrop') {
          closeEntryModal();
        } else if (event.currentTarget.id === 'settingsModalBackdrop') {
          closeSettings();
        }
      }
    }

    // Initialize modal backdrops
    window.addEventListener('DOMContentLoaded', function() {
      const entryBackdrop = document.getElementById('entryModalBackdrop');
      const settingsBackdrop = document.getElementById('settingsModalBackdrop');
      if (entryBackdrop) {
        entryBackdrop.addEventListener('click', closeModalOnBackdropClick);
      }
      if (settingsBackdrop) {
        settingsBackdrop.addEventListener('click', closeModalOnBackdropClick);
      }
    });
```

---

### Issue 2: Fix Modal Backdrops in HTML

**Location**: Find the modal backdrop divs (search for `class="modal-backdrop"`)

**Update from**:
```html
<div class="modal-backdrop" id="entryModal">
```

**Update to**:
```html
<div class="modal-backdrop" id="entryModalBackdrop">
  <div class="modal" id="entryModal">
    <!-- modal content here -->
  </div>
</div>
```

**Similar fix for settings**:
```html
<div class="modal-backdrop" id="settingsModalBackdrop">
  <div class="modal" id="settingsModal">
    <!-- modal content here -->
  </div>
</div>
```

---

### Issue 3: Fix Select Value Retrieval in `saveLesson()`

**Location**: Around line 1680 in the `saveLesson()` function

**Current problematic code**:
```javascript
category: selectedValues('categoryWrapper'),
impactLevel: getSelectedValue('impactLevelWrapper'),
// ... etc
```

**This now works** because we defined the missing functions above.

---

### Issue 4: Fix Trade Menu Item Clicks

**Location**: Around line 1590 in `renderTradeMenu()` function

**Find this line**:
```javascript
const tradeButtons = trades.map(t => `<button class="trade-item${t === selectedTradeForMenu ? ' active' : ''}" onclick="selectTradeMenu('${escapeHtml(t)}')">`
```

**This should now work** because `escapeHtml()` is now defined.

---

### Issue 5: Initialize on Page Load

**Find this section** (around line 1810):
```javascript
    populateAllSelects();
    render();
  </script>
```

**Replace with**:
```javascript
    // Load saved data first
    loadLessons();
    
    // Then populate UI
    populateAllSelects();
    render();
    
    // Set up event listeners
    const entryForm = document.getElementById('lessonForm');
    if (entryForm) {
      entryForm.addEventListener('submit', saveLesson);
    }
  </script>
```

---

## Quick Reference: All Button onclick Handlers

| Button | Handler | Status |
|--------|---------|--------|
| 📋 Register | `showScreen('register')` | ✅ Defined |
| 🏗️ Trade Lessons | `showScreen('trade')` | ✅ Defined |
| ⚙️ Settings | `openSettings()` | ✅ Defined |
| 📥 Export CSV | `exportCSV()` | ✅ Defined |
| ➕ New Lesson | `openEntryModal()` | ✅ NOW FIXED |
| Print | `print()` | ✅ Native |
| Reset Filters | `resetFilters()` | ✅ Defined |
| Save Lesson | `saveLesson(e)` | ✅ Defined |
| Delete Lesson | `deleteLesson(id)` | ✅ Defined |
| Close Modal | `closeEntryModal()` | ✅ NOW FIXED |
| Settings Add | `addOption(key)` | ✅ Defined |
| Settings Remove | `removeOption(key, value)` | ✅ Defined |

---

## Summary of Changes

**Functions Added**: 7
- `escapeHtml()`
- `persistLessons()`
- `loadLessons()`
- `getSelectedValue()`
- `selectedValues()`
- `openEntryModal()`
- `closeEntryModal()`

**Functions Fixed**: 5
- Modal backdrop event handling
- Select value retrieval
- Data persistence
- Trade menu rendering
- Page initialization
