# Prevent ScreenCapture In iOS 14

This is the code sample of how we use

UIScreen.capturedDidChangeNotification

UIApplication.userDidTakeScreenshotNotification

to detect and prevent screen capture and screenshot at iOS 14

![Prevent Screen recording](/detect-prevent-screen-recording.gif?raw=true "Prevent Screen recording")
# ScreenDetectoriOS

[https://github.com/paigeshin/ScreenDetectoriOS](https://github.com/paigeshin/ScreenDetectoriOS)

```swift
//
//  ScreenProtector.swift
//  Secret Letter
//
//  Created by innertainment on 2021/06/10.
//

import UIKit

class ScreenProtector {
    private var warningWindow: UIWindow?

    private var window: UIWindow? {
        return (UIApplication.shared.connectedScenes.first?.delegate as? SceneDelegate)?.window
    }

    func startPreventingRecording() {
        NotificationCenter.default.addObserver(self, selector: #selector(didDetectRecording), name: UIScreen.capturedDidChangeNotification, object: nil)
    }
    
    @objc private func didDetectRecording() {
        DispatchQueue.main.async {
            self.hideScreen()
            self.presentWarningWindow()
        }
    }
    
    private func hideScreen() {
        if UIScreen.main.isCaptured {
            window?.isHidden = true
        } else {
            window?.isHidden = false
        }
    }
    
    private func presentWarningWindow() {
        // Remove exsiting
        warningWindow?.removeFromSuperview()
        warningWindow = nil

        guard let frame = window?.bounds else { return }

        // Warning label
        let label = UILabel(frame: frame)
        label.numberOfLines = 0
        label.font = UIFont.boldSystemFont(ofSize: 40)
        label.textColor = .white
        label.textAlignment = .center
        label.text = "Screen recording is not allowed at our app!"

        // warning window
        var warningWindow = UIWindow(frame: frame)

        let windowScene = UIApplication.shared
            .connectedScenes
            .first {
                $0.activationState == .foregroundActive
            }
        if let windowScene = windowScene as? UIWindowScene {
            warningWindow = UIWindow(windowScene: windowScene)
        }

        warningWindow.frame = frame
        warningWindow.backgroundColor = .black
        warningWindow.windowLevel = UIWindow.Level.statusBar + 1
        warningWindow.clipsToBounds = true
        warningWindow.isHidden = false
        warningWindow.addSubview(label)

        self.warningWindow = warningWindow

        UIView.animate(withDuration: 0.15) {
            label.alpha = 1.0
            label.transform = .identity
        }
        warningWindow.makeKeyAndVisible()
    }
    

    func startPreventingScreenshot() {
        NotificationCenter.default.addObserver(self, selector: #selector(didDetectScreenshot), name: UIApplication.userDidTakeScreenshotNotification, object: nil)
    }

    @objc private func didDetectScreenshot() {
        print("detect screenshot")
        DispatchQueue.main.async {
            self.hideScreen()
            self.presentWarningWindow()
//            self.grandAccessAndDeleteTheLastPhoto()
        }
    }
    
}
```

```swift
private let screenProtector = ScreenProtector()
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        //MARK: - Prevent User From Taking Screenshot or Recording
        screenProtector.startPreventingRecording()
        screenProtector.startPreventingScreenshot()
        
        return true
    }
```