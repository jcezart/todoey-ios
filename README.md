<div align="center">

# ✅ Todoey

**A native iOS app to manage tasks by category — built with UIKit and Core Data**

![Swift](https://img.shields.io/badge/Swift-100%25-FA7343?style=for-the-badge&logo=swift&logoColor=white)
![UIKit](https://img.shields.io/badge/UIKit-UI-1A73E8?style=for-the-badge&logo=apple&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-16%2B-000000?style=for-the-badge&logo=apple&logoColor=white)
![Core Data](https://img.shields.io/badge/Core_Data-Persistence-FF6F00?style=for-the-badge&logo=apple&logoColor=white)
![MVC](https://img.shields.io/badge/Architecture-MVC-6A0DAD?style=for-the-badge)

</div>

---

## 📖 About

**Todoey** is a native iOS task manager built entirely in Swift with UIKit. It lets users organize their to-dos into named categories and manage items within each — tracking title and completion status. All data is stored locally using **Core Data**, and the interface follows the classic **MVC** pattern with UITableView, search bar, and swipe-to-delete.

Built from scratch as a complete learning project, focused on understanding data persistence strategies in iOS — from UserDefaults all the way to Core Data.

---

## ✨ Features

- 🗂️ **Task categories** — Create and delete named groups to organize your to-dos
- ✅ **Item management** — Add and delete tasks within each category, with a checkmark toggle for completion
- 🔍 **Search bar** — Filter items in real time within the active category using `NSPredicate`
- 👆 **Swipe-to-delete** — Remove items and categories with a native swipe gesture
- 💾 **Local persistence** — Full offline support via Core Data — no account or internet required
- 🏗️ **MVC architecture** — Clean separation between data model, view controllers, and storyboard views

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Language | Swift 5 |
| UI Framework | UIKit |
| Architecture | MVC |
| Persistence | Core Data (`NSPersistentContainer`) |
| Interface | Storyboard |
| Async / Threading | Main thread context isolation (Swift 6 safe) |
| Dependencies | None — no CocoaPods, SPM, or Carthage |

---

## 🔍 Key Implementation Details

### Core Data Stack

The persistent container is initialized in `AppDelegate` and the managed object context is passed down to each view controller — keeping a single source of truth for all database operations.

```swift
// AppDelegate.swift
lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentContainer(name: "TodoeyModel")
    container.loadPersistentStores { _, error in
        if let error { fatalError("Core Data load failed: \(error)") }
    }
    return container
}()

func saveContext() {
    let context = persistentContainer.viewContext
    if context.hasChanges {
        try? context.save()
    }
}
```

### NSFetchRequest with NSPredicate

Items are filtered per category using `NSPredicate`, and a search bar further narrows results by title — all through Core Data fetch requests with no manual filtering in memory.

```swift
func loadItems(with request: NSFetchRequest<Item> = Item.fetchRequest(), predicate: NSPredicate? = nil) {
    let categoryPredicate = NSPredicate(format: "parentCategory.name MATCHES %@", selectedCategory!.name!)

    if let additionalPredicate = predicate {
        request.predicate = NSCompoundPredicate(andPredicateWithSubpredicates: [categoryPredicate, additionalPredicate])
    } else {
        request.predicate = categoryPredicate
    }

    do {
        itemArray = try context.fetch(request)
    } catch {
        print("Error fetching data: \(error)")
    }

    tableView.reloadData()
}
```

### Search Bar Integration

Filtering is wired directly into `UISearchBarDelegate` — each keystroke generates a new fetch request with a `[cd]` case-insensitive predicate.

```swift
func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
    if searchText.isEmpty {
        loadItems()
    } else {
        let predicate = NSPredicate(format: "title CONTAINS[cd] %@", searchText)
        loadItems(with: Item.fetchRequest(), predicate: predicate)
    }
}
```

### Cell Reuse — Checkmark Fix

UITableView reuses cells, which causes stale checkmarks to appear when scrolling. Each cell is explicitly reset on `cellForRowAt` before applying the current item's state.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "ToDoItemCell", for: indexPath)
    let item = itemArray[indexPath.row]

    cell.textLabel?.text = item.title
    // Reset first — then apply state. Avoids stale checkmarks on reuse.
    cell.accessoryType = item.done ? .checkmark : .none

    return cell
}
```

### Swift 6 Concurrency — Main Thread Safety

All Core Data context access is confined to the main thread, avoiding data race warnings introduced with Swift 6 strict concurrency checking.

```swift
// Context operations always on main thread
DispatchQueue.main.async {
    self.context.perform {
        // safe fetch / save here
    }
}
```

---

## 🏗️ Project Structure

```
Todoey/
├── Todoey.xcodeproj
└── Todoey/
    ├── AppDelegate.swift               # Core Data stack + context setup
    ├── Controllers/
    │   ├── CategoryViewController.swift   # Category list — add, delete, navigate
    │   └── TodoListViewController.swift   # Item list — add, toggle, search, delete
    ├── Models/
    │   ├── Category+CoreDataClass.swift   # NSManagedObject subclass for Category
    │   ├── Item+CoreDataClass.swift       # NSManagedObject subclass for Item
    │   └── TodoeyModel.xcdatamodeld      # Core Data model (Category ↔ Item)
    └── Views/
        └── Main.storyboard               # Navigation + TableView layout
```

---

## 🚀 Getting Started

### Prerequisites

- Xcode 15+
- iOS 16+ simulator or device
- No additional setup — zero external dependencies

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/jcezart/todoey-ios.git
```

2. **Open in Xcode**
```bash
open Todoey.xcodeproj
```

3. **Build and run**

Select a simulator or connected device and press `▶ Run`. No CocoaPods install or SPM resolution required.

---

## 📚 Concepts Practiced

- **Core Data** — `NSPersistentContainer`, `NSManagedObject`, `NSFetchRequest`, `NSPredicate`, `NSCompoundPredicate`
- **UITableView** — delegate/datasource pattern, cell reuse lifecycle, swipe-to-delete
- **MVC architecture** — separating persistence logic from view controllers
- **UISearchBarDelegate** — real-time filtering with Core Data predicates
- **Swift 6 concurrency** — main thread context isolation to prevent data race warnings
- **Storyboard navigation** — `UINavigationController` with `performSegue` and `prepare(for:sender:)`

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

<div align="center">
  Made with ❤️ and Swift · <a href="https://github.com/jcezart">@jcezart</a>
</div>
