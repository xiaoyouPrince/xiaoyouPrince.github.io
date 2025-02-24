---
title: iOS Keychain 详解：安全存储敏感数据的利器
auther: xiaoyouPrince
date: 2025-02-24 14:57:38 +0800
categories: iOS 零碎
tags: keyChain
layout: post
---

<!--
# iOS Keychain 详解：安全存储敏感数据的利器
-->

在 iOS 开发中，安全存储敏感数据（如密码、密钥、证书等）至关重要。Keychain 是 iOS 提供的一种安全存储机制，可以帮助开发者轻松实现这一目标。本文将详细介绍 iOS Keychain 的相关内容，包括其特点、使用场景、API 使用方法、Info.plist 相关说明，以及一些最佳实践。

## 什么是 Keychain

Keychain 是 iOS 提供的一种安全存储机制，用于存储敏感信息，例如密码、证书、密钥等。Keychain 中的数据经过加密存储，并且与特定的应用程序或应用程序组关联，从而提供了较高的安全性。

### Keychain 的特点

- 安全性： Keychain 中的数据经过加密存储，即使设备被破解，也难以获取其中的敏感信息。
- 持久性： Keychain 中的数据不会因为应用程序被删除而丢失，除非用户手动清除或重置设备。
- 共享性： 同一个开发者账号下的应用程序可以共享 Keychain 中的数据，方便实现应用之间的数据共享。

### Keychain 的使用场景

- 存储用户密码： 用于存储用户的登录密码，避免明文存储在应用程序中。
- 存储加密密钥： 用于存储加密和解密数据所需的密钥。
- 存储证书： 用于存储应用程序的数字证书，用于身份验证和数据加密。
- 存储应用内购买凭据： 用于存储用户在应用内购买的凭据，防止用户重复购买。
- 实现应用间数据共享： 用于在同一个开发者账号下的多个应用程序之间共享数据。

### Keychain API 详解

iOS 提供了 Security 框架，用于访问 Keychain。以下是一些常用的 Keychain API：

- SecItemAdd： 用于向 Keychain 中添加新的 Item。
- SecItemCopyMatching： 用于从 Keychain 中检索 Item。
- SecItemUpdate： 用于更新 Keychain 中已存在的 Item。
- SecItemDelete： 用于从 Keychain 中删除 Item。

### Keychain 参数详解

Keychain 存储和检索数据时，本质上是在操作一个字典。这个字典的 key 是预定义的常量，value 是你要存储或查询的数据。以下是一些核心参数：

- kSecClass： 指定 Keychain 中存储的 Item 类型（例如：kSecClassGenericPassword、kSecClassInternetPassword 等）。
- kSecAttrService： 指定 Item 所属的服务名称，用于区分不同的 Item。
- kSecAttrAccount： 指定 Item 的账户名，用于在同一个服务下区分不同的 Item。
- kSecValueData： 指定要存储的实际数据。

### Keychain 使用示例

以下是一个使用 Keychain API 存储和检索密码的示例：

```swift
import Security

// 存储密码
func storePassword(password: String, service: String, account: String) {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecValueData as String: password.data(using: .utf8)!
    ]

    let status = SecItemAdd(query as CFDictionary, nil)

    if status != errSecSuccess {
        print("Error storing password: \(status)")
    }
}

// 检索密码
func retrievePassword(service: String, account: String) -> String? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecReturnData as String: true
    ]

    var result: AnyObject?
    let status = SecItemCopyMatching(query as CFDictionary, &result)

    if status == errSecSuccess, let data = result as? Data, let password = String(data: data, encoding: .utf8) {
        return password
    } else {
        print("Error retrieving password: \(status)")
        return nil
    }
}

// 使用示例
let service = "MyService"
let account = "user123"
let password = "mysecretpassword"

storePassword(password: password, service: service, account: account)

if let retrievedPassword = retrievePassword(service: service, account: account) {
    print("Retrieved password: \(retrievedPassword)")
}
```

### Info.plist 相关说明

在使用 Keychain 存储敏感数据时，不需要在 Info.plist 文件中添加特定的描述 key。Info.plist 文件主要用于描述应用程序的配置信息，与 Keychain 数据的访问权限无关。

### Keychain 最佳实践

- 使用 Keychain Access Groups： 如果需要在多个应用之间共享 Keychain 数据，可以使用 Keychain Access Groups。
- 设置合适的访问权限： 使用 kSecAttrAccessible 设置合适的访问权限，以确保只有授权的应用程序可以访问敏感数据。
- 处理错误： 在使用 Keychain API 时，需要处理可能出现的错误，例如存储失败、检索失败等。
- 避免存储明文密码： 尽量避免在 Keychain 中存储明文密码，可以使用哈希值或加密后的密码进行存储。
- 用户隐私： 在应用描述中说明应用使用 Keychain 存储哪些数据，以及用途，以提高用户信任感。

## Swift 高级封装

为了简化 Keychain 的使用，我们可以对其进行高级封装，提供简洁易用的接口。以下是一个封装示例：

```swift
import Security

class KeychainManager {

    static let shared = KeychainManager()

    // MARK: - 通用密码

    func storePassword(password: String, service: String, account: String) -> Bool {
        return store(value: password.data(using: .utf8)!, service: service, account: account, itemClass: kSecClassGenericPassword)
    }

    func retrievePassword(service: String, account: String) -> String? {
        guard let data = retrieve(service: service, account: account, itemClass: kSecClassGenericPassword) else { return nil }
        return String(data: data, encoding: .utf8)
    }

    func updatePassword(password: String, service: String, account: String) -> Bool {
        return update(value: password.data(using: .utf8)!, service: service, account: account, itemClass: kSecClassGenericPassword)
    }

    func deletePassword(service: String, account: String) -> Bool {
        return delete(service: service, account: account, itemClass: kSecClassGenericPassword)
    }

    // MARK: - 密钥

    func storeKey(key: Data, service: String, account: String) -> Bool {
        return store(value: key, service: service, account: account, itemClass: kSecClassKey)
    }

    func retrieveKey(service: String, account: String) -> Data? {
        return retrieve(service: service, account: account, itemClass: kSecClassKey)
    }

    func updateKey(key: Data, service: String, account: String) -> Bool {
        return update(value: key, service: service, account: account, itemClass: kSecClassKey)
    }

    func deleteKey(service: String, account: String) -> Bool {
        return delete(service: service, account: account, itemClass: kSecClassKey)
    }

    // MARK: - App Store 购买凭证

    func storeAppStoreReceipt(receipt: Data) -> Bool {
        return store(value: receipt, service: "AppStoreReceipt", account: "receipt", itemClass: kSecClassGenericPassword)
    }

    func retrieveAppStoreReceipt() -> Data? {
        return retrieve(service: "AppStoreReceipt", account: "receipt", itemClass: kSecClassGenericPassword)
    }

    func updateAppStoreReceipt(receipt: Data) -> Bool {
        return update(value: receipt, service: "AppStoreReceipt", account: "receipt", itemClass: kSecClassGenericPassword)
    }

    func deleteAppStoreReceipt() -> Bool {
        return delete(service: "AppStoreReceipt", account: "receipt", itemClass: kSecClassGenericPassword)
    }

    // MARK: - 证书

    func storeCertificate(certificate: Data, service: String, account: String) -> Bool {
        return store(value: certificate, service: service, account: account, itemClass: kSecClassCertificate)
    }

    func retrieveCertificate(service: String, account: String) -> Data? {
        return retrieve(service: service, account: account, itemClass: kSecClassCertificate)
    }

    func updateCertificate(certificate: Data, service: String, account: String) -> Bool {
        return update(value: certificate, service: service, account: account, itemClass: kSecClassCertificate)
    }

    func deleteCertificate(service: String, account: String) -> Bool {
        return delete(service: service, account: account, itemClass: kSecClassCertificate)
    }

    // MARK: - 底层方法

    private func store(value: Data, service: String, account: String, itemClass: CFString) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: itemClass,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecValueData as String: value
        ]

        let status = SecItemAdd(query as CFDictionary, nil)
        return status == errSecSuccess
    }

    private func retrieve(service: String, account: String, itemClass: CFString) -> Data? {
        let query: [String: Any] = [
            kSecClass as String: itemClass,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecReturnData as String: true
        ]

        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)

        if status == errSecSuccess, let data = result as? Data {
            return data
        }
        return nil
    }

    private func update(value: Data, service: String, account: String, itemClass: CFString) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: itemClass,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account
        ]

        let attributes: [String: Any] = [
            kSecValueData as String: value
        ]

        let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)
        return status == errSecSuccess
    }

    private func delete(service: String, account: String, itemClass: CFString) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: itemClass,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account
        ]

        let status = SecItemDelete(query as CFDictionary)
        return status == errSecSuccess
    }
}
```

### 使用示例

```swift
// 存储密码
KeychainManager.shared.storePassword(password: "mysecretpassword", service: "MyService", account: "user123")

// 检索密码
if let password = KeychainManager.shared.retrievePassword(service: "MyService", account: "user123") {
    print("Password: \(password)")
}

// 更新密码
KeychainManager.shared.updatePassword(password: "newpassword", service: "MyService", account: "user123")

// 删除密码
KeychainManager.shared.deletePassword(service: "MyService", account: "user123")

// 其他类型的数据操作类似
```

### 关键点

- 底层方法： 使用 store、retrieve、update 和 delete 等底层方法封装了 Keychain 操作，简化了代码。
- 泛型支持： 可以根据需要扩展此类，添加对其他类型数据的支持。
- 错误处理： 方法返回 Bool 值表示操作是否成功，调用者可以根据返回值进行错误处理。

### 注意事项

在存储敏感数据时，请考虑使用更高级的加密技术，例如使用 CommonCrypto 框架进行加密。
请务必妥善保管你的密钥和证书，避免泄露。

## 多应用共享 Keychain 数据

在 iOS 中，如果你想让同一开发者账号组内的多个应用共享 Keychain 数据，你需要进行一些配置和操作。以下是详细的步骤：

### 1. 启用 Keychain Sharing

- 在你的 Xcode 项目中，打开 Target 设置。
- 选择 "Signing & Capabilities" 选项卡。
- 点击 "+ Capability" 按钮，然后搜索并添加 "Keychain Sharing" 功能。

### 2. 配置 Keychain Access Group

- 在 "Keychain Sharing" 设置中，你会看到一个或多个 "Keychain Access Groups"。
- 点击 "+" 按钮添加一个新的 Access Group。
- Access Group 的名称需要符合特定的格式，通常以 group. 开头，例如 `group.com.example.sharedkeychain`。

> 
> **重要提示**： 
> 
> 所有需要共享 Keychain 数据的应用都必须使用相同的 Access Group 名称。

### 3. 在代码中使用 Access Group

在你的 Swift 代码中，当你存储或检索 Keychain 数据时，需要指定 Access Group。以下是示例代码：

```swift
import Security

// 存储数据
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrService as String: "MyService",
    kSecAttrAccount as String: "user123",
    kSecValueData as String: "mysecretpassword".data(using: .utf8)!,
    kSecAttrAccessGroup as String: "group.com.example.sharedkeychain" // 指定 Access Group
]

let status = SecItemAdd(query as CFDictionary, nil)

// 检索数据
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrService as String: "MyService",
    kSecAttrAccount as String: "user123",
    kSecReturnData as String: true,
    kSecAttrAccessGroup as String: "group.com.example.sharedkeychain" // 指定 Access Group
]

var result: AnyObject?
let status = SecItemCopyMatching(query as CFDictionary, &result)
```

### 4. 确保应用属于同一 App Group

除了配置 Keychain Access Group，你还需要确保所有需要共享 Keychain 数据的应用都属于同一个 App Group。你可以在 Xcode 的 "Signing & Capabilities" 设置中配置 App Groups。

### 注意事项

- 安全性： 共享 Keychain 数据存在一定的安全风险，请仔细评估并采取必要的安全措施。
- 用户隐私： 在应用描述中说明应用使用 Keychain 共享哪些数据，以及用途，以提高用户信任感。
- Entitlements 文件： 确保你的 Entitlements 文件中包含了正确的 Keychain Sharing 和 App Groups 配置。

请务必谨慎使用 Keychain 共享功能，并确保采取必要的安全措施。

## 总结

Keychain 是 iOS 提供的一种强大的安全存储机制，可以帮助开发者安全地存储敏感数据。通过合理使用 Keychain API，并遵循最佳实践，可以有效地保护用户隐私，提高应用安全性。

