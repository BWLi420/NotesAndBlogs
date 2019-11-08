> 在日常的程序开发中，我们不仅会使用用户的账号密码进行数据的管理，在实际情况下，对于用户通信录的获取也尤为重要，基于通信录可以扩展应用程序的使用范围。

# 通信录的开发

- 通信录开发主要是获取用户手机中的联系人
- 通过获取用户的通信录，可以在应用中添加好友等

# 访问用户通信录的方式

- 在 iOS9 之前有 2 个框架可以访问用户的通讯录
  - AddressBookUI.framework
    - 提供了联系人列表界面、联系人详情界面、添加联系人界面等
    - 一般用于选择联系人
  - AddressBook.framework
    - 纯 C 语言的 API，仅仅是获得联系人数据
    - 没有提供 UI 界面展示，需要自己搭建联系人展示界面
    - 里面的数据类型大部分基于 Core Foundation 框架
- 在 iOS9 之后，也有 2 个框架可以访问用户的通讯录
  - ContactsUI.framework
    - 对应 AddressBookUI.framework
  - Contacts.framework
    - 对应 AddressBook.framework

## 一、AddressBookUI 的使用

- 使用步骤
  - 创建选择联系人控制器
  - 设置代理
  - 实现代理方法（在代理方法中拿到用户选择的联系人）
  - 弹出控制器

  ```swift
  //1.创建联系人选择控制器
  let ppnc = ABPeoplePickerNavigationController()
  //2.设置代理
  ppnc.peoplePickerDelegate = self
  //3.弹出控制器
  present(ppnc, animated: true, completion: nil)
  ```
  实现代理方法

    ```swift
    extension ViewController: ABPeoplePickerNavigationControllerDelegate {
        /// 用户选中了某一个联系人
        ///
        /// - Parameters:
        ///   - peoplePicker: 联系人选择器
        ///   - person: 选中的联系人
        func peoplePickerNavigationController(_ peoplePicker: ABPeoplePickerNavigationController, didSelectPerson person: ABRecord) {
            //获取联系人姓名
            guard let firstName = ABRecordCopyValue(person, kABPersonFirstNameProperty).takeUnretainedValue() as? String else { return } //姓
            guard let lastName = ABRecordCopyValue(person, kABPersonLastNameProperty).takeUnretainedValue() as? String else { return } //名
            print("姓名: ", firstName, lastName)
            
            //获取联系人电话
            let phones = ABRecordCopyValue(person, kABPersonPhoneProperty).takeUnretainedValue() as ABMultiValue
            
            let count = ABMultiValueGetCount(phones)
            for i in 0..<count {
                
                let phoneLable = ABMultiValueCopyLabelAtIndex(phones, i).takeUnretainedValue() as String
                guard let phoneValue = ABMultiValueCopyValueAtIndex(phones, i).takeUnretainedValue() as? String else { return }
                
                print(phoneLable + " 电话: " + phoneValue)
            } 
        }
    }
    ```

## 二、AddressBook 的使用

![iOS9之前无UI](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtu5aqgag30i70bf0vc.gif)

- 使用步骤
  - 获取用户的授权
    - 获取授权状态
    - 如果用户是未决定状态，则请求授权
  - 获取联系人信息
    - 获取授权状态：如果是已经授权，则获取联系人信息
    - 创建通信录对象，获取通信录中所有的联系人
    - 遍历所有的联系人，获取联系人信息

- 注意：获取用户授权需要在启动的时候就询问用户

  ```swift
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
          
  	//获取用户的授权状态
      let status = ABAddressBookGetAuthorizationStatus()
          
      //判断授权状态（只有未确定状态才请求）
      if status == .notDetermined {
          //创建通信录对象
          let addressBook = ABAddressBookCreate().takeUnretainedValue()
          //请求授权
          ABAddressBookRequestAccessWithCompletion(addressBook, {
              (flag: Bool, error: CFError?) in
                  
              if flag {
                  print("授权成功")
              }else {
                  print("授权失败")
              }
          })
      }
          
      return true
  }
  ```

- 获取联系人信息

  ```swift
  //获取用户的授权状态
  let status = ABAddressBookGetAuthorizationStatus()
  //判断是否已经授权
  guard status == .authorized else {
      return
  }
          
  //创建通信录对象
  let addressBook = ABAddressBookCreate().takeUnretainedValue()
  //从对象中拷贝所有的联系人
  let peopleArray = ABAddressBookCopyArrayOfAllPeople(addressBook).takeUnretainedValue()
          
  //遍历数组，获取每一个联系人
  let count = CFArrayGetCount(peopleArray)
  for i in 0..<count {
              
      //获取指针
      let pointer = CFArrayGetValueAtIndex(peopleArray, i)
      //获取指针指向的对象
      let person = unsafeBitCast(pointer, to: ABRecord.self)
              
      //获取该联系人的姓名
      guard let firstname = ABRecordCopyValue(person, kABPersonFirstNameProperty).takeUnretainedValue() as? String else { continue }
      guard let lastname = ABRecordCopyValue(person, kABPersonLastNameProperty).takeUnretainedValue() as? String else { continue }
      print("姓名: ", firstname, lastname)
              
      //获取电话号码
      let phones = ABRecordCopyValue(person, kABPersonPhoneProperty).takeUnretainedValue() as ABMultiValue
      let phoneCount = ABMultiValueGetCount(phones)
      for j in 0..<phoneCount {
                  
          let phoneLable = ABMultiValueCopyLabelAtIndex(phones, j).takeUnretainedValue() as String
          guard let phoneValue = ABMultiValueCopyValueAtIndex(phones, j).takeUnretainedValue() as? String else { continue }
                  
          print(phoneLable + " 电话: " + phoneValue)
      }
  }
  ```

- 使用注意：若遇到权限问题，只需在 info.plist 中添加 Privacy - Contacts Usage Description 即可

## 三、ContactsUI 的使用

![iOS9之后有UI](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtu5p1ztg30i70bfq60.gif)

- 使用步骤
  - 创建选择联系人控制器
  - 设置代理
  - 实现代理方法（在代理方法中拿到用户选择的联系人）
  - 弹出控制器

  ```swift
  //创建联系人选择控制器
  let cpVC = CNContactPickerViewController()
  //设置代理
  cpVC.delegate = self      
  //弹出控制器
  present(cpVC, animated: true, completion: nil)
  ```

  实现代理方法

  ```swift
  extension ViewController: CNContactPickerDelegate {
      
      /// 选择单个联系人
      ///
      /// - Parameters:
      ///   - picker: 选择联系人控制器
      ///   - contact: 选择的联系人
      func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
          
          //获取用户姓名
          let firstName = contact.givenName
          let lastName = contact.familyName
          print("姓名: ", firstName, lastName)
          
          //获取用户电话
          let phones = contact.phoneNumbers
          for phone in phones {
              
              let phoneLabel = phone.label ?? ""
              let phoneValue = phone.value.stringValue
              
              print(phoneLabel + " 电话: " + phoneValue)
          }
      }
  }
  ```

以上代理一次只能选取一个联系人，若要一次选择多个联系人，可以使用以下的代理方法

```swift
/// 选择多个联系人
///
/// - Parameters:
///   - picker: 选择联系人控制器
///   - contact: 选择的联系人数组
func contactPicker(_ picker: CNContactPickerViewController, didSelect contacts: [CNContact]) {       
// 代码      
}s
```

## 四、Contacts 的使用

![iOS9之后无UI](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtu6n5wzg30i70bf40n.gif)

- 使用步骤
  - 获取用户的授权
    - 获取授权状态
    - 如果用户是未决定状态，则请求授权
  - 获取联系人信息
    - 获取授权状态：如果是已经授权，则获取联系人信息
    - 创建通信录对象，获取通信录中所有的联系人
    - 遍历所有的联系人，获取联系人信息

- 注意：获取用户授权需要在启动的时候就询问用户

  ```swift
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
          
      //获取授权的状态
      let status = CNContactStore.authorizationStatus(for: .contacts)
          
      //判断是否是未确定状态（是则请求授权）
      if status == .notDetermined {
              
          //创建通信录对象
          let store = CNContactStore()
              
          //请求授权
          store.requestAccess(for: .contacts, completionHandler: {
              (flag: Bool, error: Error?) in
                  
              if flag {
                  print("授权成功")
              }else {
                  print("授权失败")
              }
          })
      }
          
      return true
  }
  ```

- 获取联系人信息

  ```swift
  //获取授权状态
  let status = CNContactStore.authorizationStatus(for: .contacts)
  //判断是否已授权
  guard status == .authorized else {
      return
  }
          
  //创建通信录对象
  let store = CNContactStore()
          
  //从通信录中获取所有联系人
  let keys = [CNContactGivenNameKey as NSString,
              CNContactFamilyNameKey as NSString,
              CNContactPhoneNumbersKey as NSString]
          
  //创建请求对象
  let request = CNContactFetchRequest(keysToFetch: keys)
          
  //遍历所有联系人
  do {
      try store.enumerateContacts(with: request, usingBlock: {
          (contact: CNContact, stop: UnsafeMutablePointer<ObjCBool>) -> Void in
                  
          //获取姓名
          let firstName = contact.givenName
          let lastName = contact.familyName
          print("姓名: ", firstName, lastName)
                  
          //获取电话
          let phones = contact.phoneNumbers
          for phone in phones {
                      
              let phoneLabel = phone.label ?? ""
              let phoneValue = phone.value.stringValue
              print(phoneLabel + " 电话: " + phoneValue)
          }
      })

  } catch {
      print(error)
  }
  ```

- 使用注意：若遇到权限问题，只需在 info.plist 中添加 Privacy - Contacts Usage Description 即可

# 写在最后

本文只针对基础学习的总结，一般在我们手机通信录中的联系人的信息远远不止姓名和电话这些基础信息，还包括头像、邮箱、地址、纪念日等其他信息，这里仅以获取姓名电话为例抛砖引玉，其他的信息处理过程类似，大家触类旁通即可。

