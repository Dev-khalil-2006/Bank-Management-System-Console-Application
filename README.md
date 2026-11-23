# Bank Management System (Console Application)

A console-based bank management system written in C++ with an object-oriented design. It manages bank clients, system users with permission-based access, currency exchange rates, and money transfers. All data is stored in plain text files, so no database is required.

> **Language:** C++ (Visual Studio project, `.vcxproj`). The code uses the MSVC-specific `__declspec(property)` extension, so it builds with Visual Studio on Windows. It will not compile unchanged with GCC or Clang.

## Features

- **Client management:** add, update, delete, find, and list clients.
- **Transactions:** deposit, withdraw, transfer between clients, total balances, and a transfer log.
- **User management:** add, update, delete, find, and list system users, with per-user permissions.
- **Login and login register:** user login, and a log of every successful login.
- **Currency exchange:** list, find, and update currency rates, and a currency calculator.

## Project Structure

| Area | Files | Purpose |
|------|-------|---------|
| Entry point | `Bank Management System.cpp` | Runs the login loop in `main()`. |
| Global state | `Global.h` | Holds `CurrentUser`, the user who is logged in. |
| Base classes | `clsPerson.h` | Common person data: first name, last name, email, phone. |
| Domain models | `clsBankClient.h`, `clsUser.h`, `clsCurrency.h` | Business objects. Each one knows how to load, save, find, and delete itself. |
| Helpers | `clsString.h`, `clsDate.h`, `clsUtil.h`, `clsInputValidate.h` | String splitting, date/time, text encryption, and validated input. |
| Screens | `cls*Screen.h` | Console UI. Each screen is a class with a static `Show...` method. |
| Base screen | `clsScreen.h` | Draws the header and checks access permissions. |
| Data files | `Clients.txt`, `Users.txt`, `Currencies.txt`, `LoginRegister.txt`, `TransfersLog.txt` | Plain-text storage, one record per line. |

## Class Design

```
clsPerson                       (base: name, email, phone)
 ├── clsBankClient              (account number, PIN, balance, deposit/withdraw/transfer)
 └── clsUser                    (username, password, permissions bit mask)

clsScreen                       (base for all screens: header and access checks)
 └── cls...Screen               (one class per menu action)
```

- **Encapsulation:** each model keeps its fields private and exposes them through properties.
- **Inheritance:** `clsBankClient` and `clsUser` reuse the person data from `clsPerson`.
- **Persistence:** each model has private static methods that read and write its own data file. Screens never touch files directly.

## Data Format

Each record is one line. Fields are separated by `#//#`.

**Clients.txt**

```
FirstName#//#LastName#//#Email#//#Phone#//#AccountNumber#//#PinCode#//#Balance
```

**Users.txt**

```
FirstName#//#LastName#//#Email#//#Phone#//#UserName#//#EncryptedPassword#//#Permissions
```

**Permissions** is a bit mask. Each feature has a bit value, and the values are added together:

| Permission | Value |
|------------|-------|
| List clients | 1 |
| Add client | 2 |
| Delete client | 4 |
| Update client | 8 |
| Find client | 16 |
| Transactions | 32 |
| Manage users | 64 |
| Login register | 128 |
| **All** | **-1** |

## Getting Started

### Requirements

- Windows
- Visual Studio with the **Desktop development with C++** workload

### Run

1. Open `Bank Management System.slnx` in Visual Studio.
2. Build and run the project.
3. Log in with a user from `Users.txt`.

**Important:** the file paths are relative (for example `"Clients.txt"`). The program must run from the folder that contains the data files. Running the built executable from another folder will create empty data files there.

## Known Limitations

These are issues found in the current code. They are listed here so they are not hidden.

- **Missing return value:** `clsBankClient::Withdraw` returns `true` only in the failure branch. The success path does not return a value, which is undefined behavior in C++. Fix: add `return true;` after `Save()`.
- **Floating-point money:** balances use `float` and `double`. Binary floating-point cannot represent most decimal amounts exactly, so small rounding errors can accumulate. Storing amounts as integer cents, or using a decimal type, avoids this.
- **Weak password encryption:** `clsUtil::EncryptText` shifts each character by a fixed value (Caesar cipher). This is obfuscation, not encryption, and is easy to reverse.
- **Client PINs are stored in plain text** in `Clients.txt`.
- **Out-of-bounds write:** `EncryptText` and `DecryptText` loop with `i <= Text.length()`, which writes to the null terminator. The loop should use `i < Text.length()`.
- **No atomic transfers:** `Transfer` withdraws from one account and then deposits into another, with a separate file write for each step. If the program stops between the two steps, money can be lost. The transfer ignores the return value of `Withdraw`.
- **Global state in a header:** `Global.h` defines `CurrentUser` without `inline` or `extern`. This works only because one source file includes it. Adding a second `.cpp` file that includes it will cause a linker error.
- **Static initialization order:** `CurrentUser` is initialized by reading `Users.txt` before `main()` starts, which depends on the current working directory.
- **Missing `break`:** in `clsBankClient::Save()` and `clsUser::Save()`, the `EmptyMode` case falls through to `UpdateMode`. It is harmless today, but it is easy to break.
- **Unused functions:** `clsPerson::SendEmail`, `SendFax`, and `SendSMS` are empty stubs.

## Stray Files

- `0` is a leftover file created by the `system("pause>0")` command. It is not part of the application and can be deleted.
- `.DS_Store` is a macOS system file. Add it to `.gitignore`.

## Future Improvements

- Replace text files with a database (for example SQL Server) and move data access into a separate repository layer.
- Store money as integer cents or a decimal type.
- Hash passwords and PINs with a proper algorithm instead of the Caesar cipher.
- Make transfers atomic, so both balance updates succeed or neither does.
- Fix the issues listed under Known Limitations.
