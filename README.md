# Warning
## Description
Firebase의 Cloud Messaging을 이용해서 원격으로 알림을 발송하느 프로젝트이다.
원격으로 content 내용을 전송하고, 앱에서는 발송된 원격 알림을 받아서 표현할 수 있도록 한다.
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
  3. 터미널로 돌아와 **pod install**을 입력하여 SDK를 설치한다.
      <img src="https://user-images.githubusercontent.com/62936197/150813084-e0ffaedb-8909-4bbf-b39d-2e036afd20ed.png" width="550" height="160"> <br>
  4. pod을 추가한 후에는 xcworkspace 파일을 이용해서 개발을 진행한다.
* Apple Developer를 통한 key 등록
## Files
>AppDelegate.swift
  * FCM에 현재 등록되어 있는 토큰을 확인한다.
    ```swift
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
        
        // 기기에서 특정에 앱에 대한 알림을 받기 위해 필요한 승인
        let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
        
        UNUserNotificationCenter.current().requestAuthorization(options: authOptions) {_, error in
            print("ERROR, Request Notification Authorization: \(error.debugDescription)")
        }
        
        application.registerForRemoteNotifications()
        
        return true
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
