---
title: iOS Storyboard 깔끔하게 사용하기
permalink: organizing-storyboards
---

Storyboard는 iOS 애플리케이션을 만들 때 UI를 만들고 관리하기 가장 쉬운 방법입니다. 하지만 애플리케이션이 점점 커지고 복잡해질수록, 많은 사람이 사용할수록 다음과 같은 문제점들이 발생하게 됩니다.

 - **해결하기 힘든 Merge Conflict**: 자동생성된 XML로 유지되는 Storyboard의 특성상 두 명 이상이 팀으로 사용하게 될 경우 Merge Conflict가 발생할 가능성이 높고, Merge Conflict가 발생할 경우 해결이 힘듭니다.
 - **Code Review의 어려움**: 위와 마찬가지의 이유로 코드리뷰가 힘듭니다.
 - **화면의 복잡함**: 애플리케이션의 규모가 커질수록 화면의 개수가 많아지고 그에 따라 다양한 요소가 많아지므로 한눈에 보기 힘들고 원하지 않는 조작을 할 가능성이 커집니다.
 - **엄청나게 느려지는 로딩속도**: 화면이 늘어나면서 로딩속도가 점점 느려져 나중에는 사용하기 힘들 정도가 됩니다.
 
 Storyboard를 View에 따라 분리함으로써 위의 많은 문제를 해결할 수 있습니다.
 
 - 뷰에 따라 Storyboard가 분리되기 때문에 동시에 같은 Storyboard를 수정할 일이 줄어들고, 그에 따라 Merge Conflict가 발생할 확률도 줄어듭니다.
 - 하나의 Storyboard가 하나의 화면만 가지고 있으므로 화면의 복잡도가 현저히 줄어듭니다.
 - 화면의 개수가 하나이므로 Storyboard를 로딩하는 속도도 빨라집니다.

![alt text](https://io-inc.github.io/geeks.blog/assets/img/storyboars.jpg "각각의 뷰 마다 Storyboard 분리")

코드리뷰는 여전히 어려운 일이지만 그 부분을 제외하면 Storyboard 사용에서 감수해야 할 많은 단점을 제거할 수 있습니다.

Storyboard 분리를 통해 얻을 수 있는 또 다른 장점은 Segue를 통한 복잡한 데이터 전달을 피할 수 있다는 것입니다.

```swift
class UsersViewController: UIViewController, UITableViewDelegate {
    
  private enum SegueIdentifier {
    static let showUserDetails = "showUserDetails"
  }
    
  var usernames: [String] = ["Marin"]
    
  func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    usernameToSend = usernames[indexPath.row]
    performSegue(withIdentifier: SegueIdentifier.showUserDetails, sender: nil)
  }
    
    
  private var usernameToSend: String?
    
  override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        
    switch segue.identifier {
      case SegueIdentifier.showUserDetails?:
            
        guard let usernameToSend = usernameToSend else {
          assertionFailure("No username provided!")
          return
        }
            
        let destination = segue.destination as! UserDetailViewController
        destination.username = usernameToSend
            
      default:
        break
    }     
  }
}
```

위의 코드에서 destination으로 username을 전달하기 위해 optional variable을 사용해야 하고 destination view를 원하는 값으로 캐스팅하는 등 복잡한 과정들이 일어나게 됩니다. 또한, sugue 가 늘어나게 되면 prepare 함수에 많은 기능이 한 번에 들어가 코드의 복잡도가 커지게 됩니다.

```swift
class UsersViewController: UIViewController, UITableViewDelegate {
    
  var usernames: [String] = ["Marin"]
    
  func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let username = usernames[indexPath.row]
    showDetail(withUsername: username)
  }
    
  private func showDetail(withUsername username: String) {
    let detail = UserDetailViewController.instance()
    detail.username = username
    navigationController?.pushViewController(detail, animated: true)
  }
}
```

반면 Storyboard를 분리하게 되면 위와 같이 훨씬 안전하고 간단하게 뷰를 이동할 수 있습니다.

Storyboard의 장단점 및 Storyboard를 분리하는 자세한 방법은 아래에서 확인할 수 있습니다.

### Storyboard의 장단점
 - http://roadfiresoftware.com/2015/03/the-pros-and-cons-of-using-storyboards/

### Storyboard분리 방법
 - https://medium.cobeisfresh.com/a-case-for-using-storyboards-on-ios-3bbe69efbdf4
 - https://medium.com/ios-os-x-development/xcode-a-better-way-to-deal-with-storyboards-8b6a8b504c06

