# E2E / Visual Spec — format & examples

`e2e-plan` writes this; `visual-verify` asserts it. One `## <route>` section per surface. Routes are app-relative (the
base URL is supplied at run time). `viewport(s)` are widths; height defaults to 800px. Write each assertion as the
strongest objective form the browser can read; reserve `paint:` for a look-and-feel judgment a value cannot pin. The
three short examples below model a static page, an interaction flow, and a keyboard/ARIA section; the Reference at the
end lists the full vocabulary.

## /account/orders

- name: orders-list
- preconditions: signed in as a customer with ≥1 order
- viewport(s): 640px, 1024px
- assertions:
    - element visible: role=heading name="Your orders"
    - list visible: role=list name="Orders" items: ["#1001", "#1002", "#1003"]
    - contains: role=list name="Orders" inside role=main
    - centered horizontally in viewport: role=heading name="Your orders"   @ 1024px

## /search

- name: product-search
- preconditions: none
- viewport(s): 1024px
- flow:
    1. navigate /search
    2. fill role=textbox name="Search" with "widget"
    3. click role=button name="Search"
    4. wait for text "Results"
    5. assert element hidden: role=progressbar   # spinner cleared
- assertions:
    - text visible: "3 results"
    - value: role=combobox name="Sort" == "relevance"
    - console: clean

## /reports

- name: sortable-table-headers
- preconditions: none
- viewport(s): 769px, 1200px
- reference: design/reports-focus.png   # keyboard-focus mockup; basis for the paint: check
- flow:
    1. navigate /reports
    2. press key Tab
    3. assert focused: role=button name="Sort by Date"   # focus landed here (document.activeElement)
    4. press key Enter   # activate the focused control via keyboard
- assertions:
    - attribute: role=columnheader name="Date" aria-sort == "ascending"
    - no attribute: role=columnheader name="Trend" aria-sort
    - style: role=button name="Sort by Date" cursor == "pointer"
    - paint: red focus ring on the keyboard-focused "Sort by Date" header   @ 769px

<!-- Reference — full vocabulary (the examples above show the common forms):
     - flow verbs: navigate / navigate back / click / fill…with / select option…with / hover / press key /
       wait for text "…" / wait for text gone "…" / wait <N>s / assert <any assertion>. `press key` acts on the focused
       element (focus it first); a toggle is a `click`, so pin its start state in `preconditions`.
     - assertions: element visible / hidden / text visible / value / attribute / no attribute / style / console /
       focused / list visible / contains / contains <edges> / same width|height / aligned <edge> / centered in viewport / paint.
     - value is the control value, not the visible label. attribute / style / console / focused are objective via
       browser_evaluate (local-only); focused = document.activeElement. paint is vision-judged look-and-feel only.
     - geometry compares bounding boxes within ±1px (`± <N>px` to override); the box excludes shadow / glow / outline.
     - append `@ <width>` to scope any assertion (or inline assert) to one breakpoint; unscoped applies to all. -->
