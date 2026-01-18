# Angular – Nested Checkbox Tree with Indeterminate State
## ❌ No Signals | ❌ No CDK Virtual Scroll | ✅ Batch Processing (Heavy Data)

This document provides a **pure Angular (RxJS-free optional)** solution for a **nested checkbox tree** that:

- Supports **N-level hierarchy**
- Handles **checked / unchecked / indeterminate** states
- Works with **heavy data loads (10k–50k nodes)**
- Uses **batch processing** to avoid UI freezes
- Uses **OnPush + minimal DOM updates**

---

## 🎯 When to Use This Approach

- Legacy Angular apps (no Signals)
- CDK not allowed / restricted
- Large permission trees, asset trees
- Performance issues due to deep recursion

---

## 🧱 Tree Node Model

```ts
export interface TreeNode {
  id: string;
  label: string;
  checked: boolean;
  indeterminate: boolean;
  children?: TreeNode[];
}
```

---

## 🧠 Core Design Principles

1. **Never traverse entire tree on click**
2. **Process only affected subtree**
3. **Batch child updates** (chunk execution)
4. **Iterative loops instead of deep recursion**
5. **Parent lookup map for O(depth) updates**

---

## 🗺 Parent Lookup Map (One-Time Cost)

```ts
parentMap = new Map<string, TreeNode | null>();

buildParentMap(nodes: TreeNode[], parent: TreeNode | null = null) {
  const stack = [...nodes];
  while (stack.length) {
    const node = stack.pop()!;
    this.parentMap.set(node.id, parent);
    if (node.children) {
      node.children.forEach(child => stack.push(child));
    }
  }
}
```

✔ Avoids recursive parent search on every toggle

---

## ⚙️ Batch Processing Utility

Processes children in **chunks** to avoid blocking UI thread.

```ts
processInBatches<T>(
  items: T[],
  batchSize: number,
  handler: (item: T) => void,
  done?: () => void
) {
  let index = 0;

  const runBatch = () => {
    const end = Math.min(index + batchSize, items.length);

    for (; index < end; index++) {
      handler(items[index]);
    }

    if (index < items.length) {
      setTimeout(runBatch, 0); // yield to UI thread
    } else {
      done?.();
    }
  };

  runBatch();
}
```

---

## 🛠 Tree Checkbox Service (Batch-Aware)

```ts
@Injectable({ providedIn: 'root' })
export class TreeCheckboxBatchService {

  toggleDownBatch(node: TreeNode, checked: boolean) {
    const stack: TreeNode[] = [node];

    while (stack.length) {
      const current = stack.pop()!;
      current.checked = checked;
      current.indeterminate = false;

      current.children && stack.push(...current.children);
    }
  }

  updateParentState(node: TreeNode) {
    if (!node.children?.length) return;

    const allChecked = node.children.every(c => c.checked);
    const noneChecked = node.children.every(c => !c.checked && !c.indeterminate);

    node.checked = allChecked;
    node.indeterminate = !allChecked && !noneChecked;
  }
}
```

---

## 🧩 Component – Batch-Based Toggle Logic

```ts
@Component({
  selector: 'app-checkbox-tree',
  templateUrl: './tree.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class CheckboxTreeComponent {

  @Input() treeData!: TreeNode[];
  parentMap = new Map<string, TreeNode | null>();

  constructor(private svc: TreeCheckboxBatchService) {}

  ngOnInit() {
    this.buildParentMap(this.treeData);
  }

  onToggle(node: TreeNode, checked: boolean) {
    // 1️⃣ Downward batch update
    this.svc.toggleDownBatch(node, checked);

    // 2️⃣ Upward propagation (O(depth))
    let parent = this.parentMap.get(node.id);
    while (parent) {
      this.svc.updateParentState(parent);
      parent = this.parentMap.get(parent.id);
    }
  }

  trackById(_: number, node: TreeNode) {
    return node.id;
  }
}
```

---

## 📄 Template (Lightweight & Recursive)

```html
<ul>
  <li *ngFor="let node of treeData; trackBy: trackById">

    <input
      type="checkbox"
      [checked]="node.checked"
      [indeterminate]="node.indeterminate"
      (change)="onToggle(node, $event.target.checked)"
    />

    {{ node.label }}

    <app-checkbox-tree
      *ngIf="node.children"
      [treeData]="node.children">
    </app-checkbox-tree>

  </li>
</ul>
```

---

## 🚀 Heavy Data Optimizations (Without CDK)

### ✅ Lazy Child Rendering

```html
*ngIf="node.expanded"
```

### ✅ Expand on Demand
- Render children **only when expanded**

### ✅ Batch Select All

```ts
this.processInBatches(allNodes, 500, n => n.checked = true);
```

### ✅ Avoid ChangeDetectorRef.detectChanges()
- Let Angular batch DOM updates naturally

---

## 🧪 Edge Cases Covered

- Partial child selection
- Deep nesting (N levels)
- Parent with zero children
- Large subtree select/deselect
- Mixed states recovery

---

## ⏱ Performance Characteristics

| Operation | Cost |
|--------|------|
| Toggle subtree | O(subtree size) |
| Parent update | O(depth) |
| Full tree scan | ❌ Never |

---

## 🏢 Enterprise Best Practices

- Persist selected IDs in `Set<string>`
- Defer heavy operations using batching
- Throttle bulk selection
- Unit-test tree logic separately

---

## ✅ Summary

✔ No Signals  
✔ No CDK Virtual Scroll  
✔ Batch processing prevents UI freeze  
✔ Scales to **tens of thousands of nodes**  
✔ Clean, interview-ready design

---

## ➕ Want Next?

- Batch + RxJS version
- Unit tests for N-level tree
- Grid + tree combined selection
- Backend-driven partial hydration
- Performance benchmarking guide

Tell me 👍

