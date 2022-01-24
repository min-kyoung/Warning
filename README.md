# Warning
## Description
Firebase의 Cloud Messaging을 이용해서 원격으로 알림을 발송하는 프로젝트이다. <br>
원격으로 content 내용을 전송하고, 앱에서는 발송된 원격 알림을 받아서 표현할 수 있도록 한다. <br>
<img src="https://user-images.githubusercontent.com/62936197/150818679-b2f68beb-bc97-4f8c-b2fd-b5930f17c971.jpeg" width="250" height="70">
## Prerequisite
* Firebase를 이용하기 위해 프로젝트 추가를 한다.
  1. https://firebase.google.com 에서 새 프로젝트를 생성한다.
  2. 콘솔 앱 프로젝트가 추가되었다면 ios 앱 추가를 선택한다. <br>
     이 때, iOS의 번들 ID에는 Xcode 프로젝트 파일의 Bundle Identifier를 입력한다.
  3. GoogleService-info.plist를 다운받아 Xcode 프로젝트 파일에 추가한다.
* Firebase SDK를 CocoaPods를 이용하여 설치한다.
  1. 터미널에서 해당 프로젝트 경로로 이동한 후 **pod init**을 입력하여 Podfile을 생성한다. <br>
     <img src="https://user-images.githubusercontent.com/62936197/150813076-a87cdb32-17d7-4989-a72a-da5ce073a6bd.png" width="250" height="150"> <br>
  2. Podfile을 열어서 **# Pods for Warning** 아래에 두 가지를 추가한 후 저장한다.
     ```swift
     pod 'Firebase/Analytics'
     pod 'Firebase/Messaging'
     ```
  3. 터미널로 돌아와 **pod install**을 입력하여 SDK를 설치한다. <br>
     <img src="https://user-images.githubusercontent.com/62936197/150813084-e0ffaedb-8909-4bbf-b39d-2e036afd20ed.png" width="550" height="160"> <br>
  4. pod을 추가한 후에는 xcworkspace 파일을 이용해서 개발을 진행한다.

* APNs에 키 등록을 위해 Apple Developer를 통한 key 발급을 한다.
  1. XCode 프로젝트에서 **TARGETS > Signing & Capabilities**를 선택한 후 Capability 추가에서 **Push Notifications**를 선택한다.
     <img src="https://user-images.githubusercontent.com/62936197/150823413-bd91bd2c-8d5c-4a71-947a-cbf98f0ced18.png" width="400" height="200"> 
  2. **https://developer.apple.com > Account > Overview > Certificates, Identifiers & Profiles**항목으로 들어간 후 Keys 탭에서 **Create a key**를 한다.<br>
     <img src="https://user-images.githubusercontent.com/62936197/150820951-7eda1d4d-3198-48fc-ac50-ecdb32e5676a.png" width="550" height="160">
  3. 좌측에 Keys 탭을 선택한 후 +버튼을 통해 Key를 생성한다. 이 때, APNs를 활성화 해준다. <br>
     <img src="https://user-images.githubusercontent.com/62936197/150824382-c5c371b1-d8ab-4ecf-a1d2-7b088e0e15bb.png" width="550" height="160">
  4. key가 생성되었으면 Download를 한다. (Download는 한 번만 가능하다.)
 
* Firebase에 key값을 등록한다.
  1. Firebase의 프로젝트 설정에서 클라우드 메시징 탭으로 들어가 **Apple 앱 구성** 항목에 다운로드한 Key를 업로드한다. 
     <img src="https://user-images.githubusercontent.com/62936197/150825944-3c448187-674f-475e-8194-6e96011565ad.png" width="550" height="160">
  2. 키 ID는 Apple Developer를 통해 발급받은 Key ID를 입력하고, 팀 ID는 **https://developer.apple.com > Account > Overview > Certificates, Identifiers & Profiles**항목으로 들어간 후 Membership 탭에서 확인한 값을 입력한다.
## Files
>AppDelegate.swift
  * FCM에 현재 등록되어 있는 토큰을 확인한다.
    ```swift
    func application(_ application: UIApplication, willFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        UNUserNotificationCenter.current().delegate = self
        return true
    }
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        FirebaseApp.configure() // Firebase 초기화
        
        Messaging.messaging().delegate = self // Messaging 초기화
        
        // FCM에 현재 등록된 토큰 확인
        Messaging.messaging().token { token, error in
            if let error = error {
                print("ERROR FCM 등록 토큰 가져오기: \(error.localizedDescription)")
            } else if let token = token {
                print("FCM 등록 토큰: \(token)")
            }
        }
        
        // 기기에서 특정 앱에 대한 알림을 받기 위해 필요한 승인
        let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
        
        UNUserNotificationCenter.current().requestAuthorization(options: authOptions) {_, error in
            print("ERROR, Request Notification Authorization: \(error.debugDescription)")
        }
        
        application.registerForRemoteNotifications()
        
        return true
    }
    ```
  * 원격으로 받은 notitication의 display 형태를 지정한다.
    ```swift
    extension AppDelegate: UNUserNotificationCenterDelegate {
        func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
            completionHandler([.list, .banner, .badge, .sound]) // 리스트, 배너, 뱃지, 사운드 설정
        }
    }
    ```
  * 토큰이 갱신되었는지 확인한다.
    ```swift
    extension AppDelegate: MessagingDelegate {
        // token이 갱신되는 시점
        // 다시 token을 받았는지 확인
        func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
            guard let token = fcmToken else { return }
            print("FCM 등록 토큰 갱신 :\(token)")
        }
    }
    ```
>SceneDelegate
  * 앱에 접속했을 때 뱃지가 사라지도록 한다.
    ```swift
    func sceneDidBecomeActive(_ scene: UIScene) {
          if UIApplication.shared.applicationIconBadgeNumber != 0 {
              UIApplication.shared.applicationIconBadgeNumber = 0
          }
    }
    ```
## Usage
* 원격 알림은 시뮬레이터에서 지원하지 않기 때문에 실제 기기를 연결하여 알람을 확인한다.
1. build에 성공하면 console에 FCM 등록 토큰 값을 확인할 수 있다. <br>
  <img src="https://user-images.githubusercontent.com/62936197/150828509-c7d34f27-ec4d-4e34-aae2-b1e3d992f6a9.png" width="350" height="80"> <br>
2. Firebase의 Cloud Messaging을 선택한 후 **Send your first message**를 클릭한다.
3. 알림 항목에서 알림 제목과 알림 텍스트를 입력한 후 메시지 전송 버튼을 누르면 FCM 토큰을 추가하라는 창이 나타나는데, 여기에 console에서 확인한 FCM 등록 토큰 값을 입력한다. <br>
   <img src="https://user-images.githubusercontent.com/62936197/150831951-e82ed653-6d11-48ba-aa34-938fced626be.png" width="350" height="80"> <br>
4. 타겟 항목에서는 현재 개발 중인 앱으로 설정하고, 예약 항목에서는 발송 시점을 지금으로 선택한다. <br>
   <img src="https://user-images.githubusercontent.com/62936197/150831962-5523b7ac-8428-49b1-b1ef-503fec5fb76d.png" width="350" height="80"> <br>
5. 추가 옵션 항목에서 알림음, iOS 배지를 사용설정 하고, 배지 수는 확인 용도로 사용하므로 임의의 값을 입력한다. <br>
   <img src="https://user-images.githubusercontent.com/62936197/150831984-9eb3dc8d-cc9c-45ee-922b-f35c38423ec5.png" width="350" height="80"> <br>
6. 검토를 누르면 아래와 같이 나타나며, 몇 초 내로 알림이 뜨고 배지까지 표현된다. <br>
   <img src="https://user-images.githubusercontent.com/62936197/150831996-9e52c4cf-59f6-4580-b984-60732bf660fb.png" width="550" height="100"> <br>
