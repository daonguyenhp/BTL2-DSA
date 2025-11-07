AVL Tree Balancing Theory: BalanceValue and Rotations (With Diagrams)

This document translates the core concepts of AVL tree balancing—using a simple BalanceValue instead of full height—into a structured explanation for implementation reference, enhanced with visual diagrams.

1. AVLNode* rotateRight(AVLNode*& node) - Detailed Example

Đây là ví dụ chi tiết về thao tác xoay phải, dựa trên mã nguồn và minh họa bạn đã cung cấp.

Initial Condition:
Node *&pos;

              |
             [P]
           /     \
          L       R
         / \     / \
        w   x   y   z


Step 1: Node *b = pos->left;
(Lưu trữ nút con trái của P vào b - đây sẽ là gốc mới sau khi xoay)

Step 2: pos->left = b->right;
(Nút con trái của P trở thành nút con phải của L - tức là x di chuyển lên làm con trái của P)

               |
        L     [P]
      /  \   /   \
     w     x      R
                 / \
                y   z


Step 3: b->right = pos;
(P trở thành nút con phải của L)

        L
      /  \ |
     w    [P]
         /   \
        x     R
             / \
            y   z


Resulting Structure (After Right Rotation):

            [L]
           /   \
          w     [P]
               /   \
              x     R
                   / \
                  y   z


2. The Theory of BalanceValue

This is the "classic" implementation. Instead of storing the full height $h$ (a large integer), we only store the difference:

$$\text{BalanceValue} = \text{Height}(\text{Right\_Subtree}) - \text{Height}(\text{Left\_Subtree})
$$  * We use three values:

* **EH (Equal High - 0):** The two subtrees are the same height.
* **LH (Left High - -1):** The LEFT subtree is 1 level taller.
* **RH (Right High - +1):** The RIGHT subtree is 1 level taller.

Every node in a valid AVL tree must only have one of these three values. If a node's balance becomes +2 or -2, it is "unbalanced" and requires rotation.

-----

## 3\. The Theory of the "Taller" Flag

Since we don't store the actual height, we cannot recalculate it. We need a way for the recursive function to "report" back to its parent node: "I just received a new node, and this has caused my entire subtree to grow 1 level taller."

This is the purpose of the `bool& taller` flag:

* `taller = true`: Tells the parent: "The subtree I am the root of JUST grew 1 level taller."
* `taller = false`: Tells the parent: "I handled it. My subtree did NOT get any taller (either it balanced itself, or I performed a rotation)."

-----

## 4\. The Theory of Updates (During Recursive Unwinding)

When the `insertHelper` function unwinds (returns from recursion) and `taller` is true, a parent node (P) looks at its current balance:

* **P is currently EH (0):**

* Insert on the left $\rightarrow$ P becomes LH (-1).
* Insert on the right $\rightarrow$ P becomes RH (+1).
In both cases, P's own subtree has grown taller. It must continue to report `taller = true` to its parent.

* **P is currently LH (-1) or RH (+1):**

* If the insert happened on the "shorter" side (e.g., P is LH, insert on the right) $\rightarrow$ P becomes EH (0).
P's subtree has balanced itself (in terms of height).
$\rightarrow$ It reports `taller = false` ("Don't worry, parent, I handled it. The total height didn't change.").

* **P is currently LH (-1) or RH (+1):**

* If the insert happened on the "taller" side (e.g., P is LH, insert on the left) $\rightarrow$ P becomes unbalanced (-2)\!
This is the **TRIGGER** for the `balanceLeft` or `balanceRight` functions.
After the rotation is complete, the tree is balanced (and the total height is the same as before the insertion).
$\rightarrow$ It reports `taller = false`.

-----

## 5\. The Theory of Rotations (The balance functions)

This is the core of the problem.

### A. `balanceLeft` Function (Handling Left-side Imbalance)

**Trigger:** Called when P is LH (-1) and its left child L reports `taller = true`.
**State:** P is now unbalanced (-2). We must check the balance of L.

* **Case 1: L-L (Straight-line Imbalance)**
**Situation:** L also has a balance of LH (-1).
**Diagram (Before):**

```
[P] (LH, balance = -2 due to insert)
/
[L] (LH)
/
(new insert)
```

**Action:** Single **Right Rotation** at P.
**Diagram (After Right Rotation):**

```
[L] (EH)
/   \
/     \
(new)  [P] (EH)
/   \
(x)   (R)
```

**Balance Update:**

* P (old root) $\rightarrow$ EH (0)
* L (new root) $\rightarrow$ EH (0)
Both become balanced.

* **Case 2: L-R (Zig-zag Imbalance)**
**Situation:** L has a balance of RH (+1).
**Diagram (Before):**

```
[P] (LH, balance = -2 due to insert)
/
[L] (RH)
\
[x]  <-- 'x' is where new insert happened or is part of subtree
```

**Action:** Double (Left-Right) Rotation. (First Left Rotation at L, then Right Rotation at P). Node `x` will become the new root.
**Diagram (After Left Rotation at L):**

```
      [P]
      /
    [x]
   /   \
  [L]   (new)
 /
(w)
```

**Diagram (After Right Rotation at P):**

```
      [x] (EH)
     /   \
    /     \
  [L]     [P]
  /       /   \
(w)     (new) (R)
```

**Balance Update:**

* x (new root) always becomes EH (0).
* The new balance of P and L (now children of x) depends on the old balance of x (before the rotation):
* If old x was LH (-1): new P is RH (+1), new L is EH (0).
* If old x was RH (+1): new P is EH (0), new L is LH (-1).
* If old x was EH (0) (this happens when x is the newly inserted node): Both new P and L are EH (0).

### B. `balanceRight` Function (Handling Right-side Imbalance)

This is the perfect mirror image of `balanceLeft`.
**Trigger:** Called when P is RH (+1) and its right child R reports `taller = true`.
**State:** P is now unbalanced (+2). We must check the balance of R.

* **Case 1: R-R (Straight-line Imbalance)**
**Situation:** R also has a balance of RH (+1).
**Diagram (Before):**

```
[P] (RH, balance = +2 due to insert)
  \
  [R] (RH)
    \
   (new insert)
```

**Action:** Single **Left Rotation** at P.
**Diagram (After Left Rotation):**

```
      [R] (EH)
     /   \
    /.     \
  [P]    (new)
  /   \
(L)  (y)
```

**Balance Update:**

* P (old root) $\rightarrow$ EH (0)
* R (new root) $\rightarrow$ EH (0)

* **Case 2: R-L (Zig-zag Imbalance)**
**Situation:** R has a balance of LH (-1).
**Diagram (Before):**

```
[P] (RH, balance = +2 due to insert)
\
[R] (LH)
/
[y]  <-- 'y' is where new insert happened or is part of subtree
```

**Action:** Double (Right-Left) Rotation. (First Right Rotation at R, then Left Rotation at P). Node `y` will become the new root.
**Diagram (After Right Rotation at R):**

```
        [P]
          \
          [y]
         /   \
       (new) [R]
          \
          (z)
```

**Diagram. (After Left Rotation at P):**

```
      [y] (EH)
     /   \
    /     \
  [P]     [R]
  /   \       \
(L) (new)    (z)
```

**Balance Update:**

* y (new root) always becomes EH (0).
* The new balance of P and R (now children of y) depends on the old balance of y:
* If old y was LH (-1): new P is EH (0), new R is RH (+1).
* If old y was RH (+1): new P is LH (-1), new R is EH (0).
* If old y was EH (0) (y is the new node): Both new P and R are EH (0).

-----

## 6\. Complexity Analysis: The $\mathbf{O(\log n)}$ Insertion

The overall complexity of the `insert` operation in an AVL tree is $\mathbf{O(\log n)}$.

* **Detailed Explanation:**

1.  **Search Phase:** $O(\log n)$
Similar to a standard Binary Search Tree (BST), the `insert` function first traverses from the root to find the insertion point. Because an AVL tree is always balanced, its height ($h$) is guaranteed to be $O(\log n)$ (where $n$ is the total number of nodes).
Therefore, finding the path to the leaf takes $O(\log n)$ time.

2.  **Update & Balancing Phase (Recursive Unwinding):** This phase occurs as the function returns from recursive calls after the new node has been inserted.

* **Traversing Back Up:** The function must go back up from the parent of the newly inserted node to the root. This path also has a maximum length of $h$, i.e., $O(\log n)$.
* **Work at Each Node ($O(1)$):** At each node on the path upwards, we need to update the `BalanceValue` (based on the `taller` flag) or recalculate height and balance. Both these operations are $O(1)$ (constant time).
* **Rotation ($O(1)$):** This is the crucial point. A rotation (single or double) involves only changing a few pointers, so it takes $O(1)$ time. During an insertion, you only need to perform a maximum of **one** rotation (at the first unbalanced node encountered on the way up). After this rotation, that subtree becomes balanced, and its height does not change compared to before the insertion, meaning no further rotations are needed higher up the tree.

## 7\. Complexity Analysis

* **Conclusion:**
    The Total Complexity is the sum of: (Search Time) + (Path Update Time) + (Rotation Time).

$$
\text{Total Complexity} = O(\log n) + O(\log n) + O(1)
$$

Thus, the overall complexity of the **`insert`** function is $\mathbf{O(\log n)}$.

---

### **💻 C++ Code: Right Rotation (Conceptual Sketch)**

This is a conceptual sketch for a **Single Right Rotation**, typically applied in the **L-L (Left-Left)** imbalance case:

```cpp
/**
 * Conceptual sketch of a Single Right Rotation (used for the L-L case).
 * This function handles the physical pointer changes, but the BalanceValue
 * updates are usually handled by the calling balance function (e.g., balanceLeft).
 *
 * @param node The current root of the unbalanced subtree (P).
 * @return The new root of the balanced subtree (L).
 */
AVLNode* rotateRight(AVLNode*& node) {
    // 1. Store the new root (L), which is P's left child.
    AVLNode *b = node->left;

    // 2. Perform the rotation pivot:
    // P's new left child becomes the new root's old right subtree (x).
    node->left = b->right;

    // 3. P (the old root) becomes the right child of the new root (L).
    b->right = node;

    // 4. (Balance updates go here based on the specific case, e.g., L-L vs L-R)
    // For L-L case, we typically set both P and L to EH (0) here.
    // node->balance = EH;
    // b->balance = EH;

    // 5. Return the new root (b).
    return b;
}
```

## 8\. 🌳 Giải Thích Chi Tiết Quá Trình Xóa Node trong Cây AVL (`removeHelper`)

Hàm `removeHelper` xử lý 3 nhiệm vụ chính: **Tìm kiếm**, **Xóa** (thay thế), và **Cân bằng** (Rebalance) bằng cách lan truyền cờ `shorter` (chiều cao giảm).

---

### 1. Tìm Kiếm Node và Lan Truyền Thay Đổi

Quá trình tìm kiếm node dựa vào giá trị `key` và gọi đệ quy. Sau khi quay lại (return) từ cuộc gọi đệ quy, ta cập nhật `balance factor` (BF) nếu `shorter` là `true`.

| Trạng Thái | Điều kiện | Cập nhật BF (Nếu `shorter` = `true`) |
| :--- | :--- | :--- |
| **Node rỗng** | `node == nullptr` | Dừng đệ quy. `shorter = false`. |
| **Đi sang trái** | `key < node->key` | `node->balance` **+1** (Cây trái rút ngắn). |
| **Đi sang phải** | `key > node->key` | `node->balance` **-1** (Cây phải rút ngắn). |

---

### 2. Xử Lý Xóa Node (Khi `key == node->key`)

#### 🟢 Trường hợp 0 hoặc 1 Con (Leaf hoặc Single Child)

Node được thay thế bằng con của nó (hoặc `nullptr`), `shorter` được đặt là `true` để bắt đầu quá trình cân bằng ngược lên.

```cpp
if (node->pLeft == nullptr || node->pRight == nullptr) {
    // Sử dụng ternary operator để xử lý cả 0 và 1 con
    AVLNode* oldNode = node;
    node = (node->pLeft != nullptr) ? node->pLeft : node->pRight;
    
    shorter = true; // Chiều cao chắc chắn giảm
    size--;
    delete oldNode;
    return node;
}
```

# 💻 Giải Thích Chi Tiết Các Hàm Cốt Lõi của Cây AVL (C++14/Lambda)

File này giải thích ba hàm quan trọng trong cấu trúc dữ liệu Cây AVL: `getMinNode`, `removeHelper`, và `inorderTraversal`.

---

## 1. 🔍 Hàm Tìm Node Nhỏ Nhất (`getMinNode`)

Hàm này tìm kiếm node có key nhỏ nhất trong một cây con (subtree), hay còn gọi là **Inorder Successor** (nếu gọi từ `pRight`). Phiên bản này sử dụng `auto` (C++14) và vòng lặp tối ưu.

```cpp
/**
 * @brief Finds the node with the minimum key in the subtree rooted at 'node'.
 * @param node The root of the subtree to search.
 * @return A pointer to the node with the minimum key. Returns nullptr if the subtree is empty.
 * @Complexity O(log n)
 */
template <class K, class T>
typename AVLTree<K, T>::AVLNode* AVLTree<K, T>::getMinNode(AVLNode* node) const noexcept {
    if (node == nullptr) {
        return nullptr;
    }

    // Sử dụng 'auto' cho biến cục bộ (C++14)
    auto current = node; 
    
    // Duyệt xuống nhánh trái nhất
    while (current->pLeft != nullptr) {
        current = current->pLeft;
    }
    
    return current;
}
