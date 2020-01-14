자바스크립트에서 이메일 링크를 사용하여 Firebase에 인증하기
Firebase 인증을 사용하면 로그인 링크를 이메일로 전송해서 사용자가 바로 로그인하게 할 수 있습니다. 이 과정에서 사용자의 이메일 주소도 인증됩니다.

이메일로 로그인하는 경우 다음과 같은 많은 이점이 있습니다.

편리한 가입 및 로그인
여러 애플리케이션에서 비밀번호를 재사용할 위험이 적음: 재사용하면 아무리 보안등급이 높은 비밀번호라 해도 보안이 약화될 수 있음
사용자를 인증하는 동시에 사용자가 이메일 주소의 합법적인 소유자인지 확인하는 기능
액세스 가능한 이메일 계정만 있으면 로그인 가능: 전화번호 또는 소셜 미디어 계정 소유를 필요로 하지 않음
사용자가 모바일 기기에서 번거롭게 비밀번호를 입력하거나 기억할 필요 없이 안전하게 로그인 가능
이전에 이메일 식별자(비밀번호 또는 제휴)로 로그인한 기존 사용자는 이메일만 사용하여 로그인하도록 업그레이드 가능: 사용자가 비밀번호를 기억하지 못하더라도 비밀번호를 재설정하지 않고 계속 로그인 가능
시작하기 전에
아직 복사하지 않았다면 자바스크립트 프로젝트에 Firebase 추가에 설명된 대로 Firebase Console의 초기화 스니펫을 프로젝트에 복사합니다.

Firebase 프로젝트에서 이메일 링크 로그인 사용 설정
이메일 링크로 사용자를 로그인 처리하려면 우선 Firebase 프로젝트에서 이메일 제공업체 및 이메일 링크 로그인 방법을 사용 설정해야 합니다.

Firebase Console에서 인증 섹션을 엽니다.
로그인 방법 탭에서 이메일/비밀번호 제공업체를 사용 설정합니다. 이메일 링크 로그인을 사용하려면 이메일/비밀번호 로그인이 사용 설정되어야 합니다.
같은 섹션에서 이메일 링크(비밀번호가 없는 로그인) 로그인 방법을 사용 설정합니다.
저장을 클릭합니다.
사용자의 이메일 주소로 인증 링크 전송
이 인증 과정을 시작하려면 사용자에게 이메일 주소를 제공하도록 요청하는 인터페이스를 제시하고 sendSignInLinkToEmail을 호출하여 Firebase가 사용자의 이메일에 인증 링크를 전송하도록 요청합니다.

Firebase에 이메일 링크를 만드는 방법에 대한 지침을 제시하는 ActionCodeSettings 객체를 만듭니다. 다음 필드를 설정합니다.

url: 삽입할 딥 링크 및 함께 전달할 추가 상태. 승인된 도메인의 Firebase Console 목록에서 링크의 도메인을 허용 목록에 추가해야 하며 로그인 방법 탭(인증 -> 로그인 방법)으로 이동하여 확인할 수 있습니다.
android 및 ios: Android 또는 iOS 기기에서 로그인 링크를 열 때 사용할 앱. 모바일 앱을 통해 이메일 작업 링크를 열기 위해 Firebase 동적 링크를 구성하는 방법에 대해 자세히 알아보기
handleCodeInApp: true로 설정합니다. 다른 대역 외 이메일 작업(비밀번호 재설정 및 이메일 확인)과 달리 이 로그인 작업은 항상 앱에서 완료해야 합니다. 그 이유는 인증 과정 마지막에 사용자가 로그인하고 사용자의 인증 상태를 앱에서 유지해야 하기 때문입니다.
dynamicLinkDomain: 프로젝트에 대해 여러 개의 커스텀 동적 링크 도메인이 정의된 경우 지정된 모바일 앱을 통해 링크를 열 때 사용할 도메인을 지정합니다(예: example.page.link). 그렇지 않으면 첫 번째 도메인이 자동으로 선택됩니다.

var actionCodeSettings = {
  // URL you want to redirect back to. The domain (www.example.com) for this
  // URL must be whitelisted in the Firebase Console.
  url: 'https://www.example.com/finishSignUp?cartId=1234',
  // This must be true.
  handleCodeInApp: true,
  iOS: {
    bundleId: 'com.example.ios'
  },
  android: {
    packageName: 'com.example.android',
    installApp: true,
    minimumVersion: '12'
  },
  dynamicLinkDomain: 'example.page.link'
};

ActionCodeSettings에 대해 자세히 알아보려면 이메일 작업의 상태 전달 섹션을 참조하세요.

사용자에게 이메일 주소 입력을 요청합니다.

사용자의 이메일에 인증 링크를 전송하고 사용자가 같은 기기에서 이메일 로그인을 완료할 경우를 대비해 사용자의 이메일을 저장합니다.


firebase.auth().sendSignInLinkToEmail(email, actionCodeSettings)
  .then(function() {
    // The link was successfully sent. Inform the user.
    // Save the email locally so you don't need to ask the user for it again
    // if they open the link on the same device.
    window.localStorage.setItem('emailForSignIn', email);
  })
  .catch(function(error) {
    // Some error occurred, you can inspect the code: error.code
  });

이메일 링크로 로그인 완료
보안 문제
의도하지 않은 사용자나 기기에서 로그인 링크가 사용되는 것을 방지하기 위해 Firebase 인증에서는 로그인 과정을 완료할 때 사용자의 이메일 주소를 입력해야 합니다. 로그인하려면 이 이메일 주소가 처음에 로그인 링크를 보낸 주소와 일치해야 합니다.

링크를 요청한 기기에서 로그인 링크를 여는 사용자를 위해 이 과정을 간소화할 수 있습니다. 예를 들어 로그인 이메일을 보낼 때 localStorage 또는 쿠키를 사용하여 사용자의 이메일 주소를 로컬에 저장하면 됩니다. 그런 다음 이 이메일 주소를 사용하여 이 과정을 완료합니다. 세션 주입이 활성화될 수 있으므로 사용자의 이메일을 리디렉션 URL 매개변수에서 전달해서는 안 되며 재사용해서도 안 됩니다.

로그인이 완료되면 확인되지 않은 이전 로그인 메커니즘은 사용자에게서 모두 삭제되고 기존 세션은 무효화됩니다. 예를 들어 누군가가 이전에 같은 이메일과 비밀번호로 확인되지 않은 계정을 만든 경우 이 사용자의 비밀번호는 삭제됩니다. 명의를 도용해 확인되지 않은 계정을 만들었던 사람이 이 이메일과 비밀번호로 다시 로그인하는 것을 방지하기 위해서입니다.

또한 중개 서버에서 링크를 가로채지 않도록 프로덕션 시 HTTPS URL을 사용해야 합니다.

웹 페이지에서 로그인 완료
이메일 링크 딥 링크의 형식은 대역 외 이메일 작업에 사용된 형식과 같습니다. 이러한 작업에는 이메일 확인, 비밀번호 재설정 및 이메일 변경 취소 등이 있습니다. Firebase 인증에서는 이메일 링크를 사용한 로그인 링크인지 확인하는 데 isSignInWithEmailLink API를 제공하여 간편하게 이 확인을 처리합니다.

방문 페이지에서 로그인을 완료하려면 사용자의 이메일과 1회용 코드가 포함된 실제 이메일 링크를 사용하여 signInWithEmailLink를 호출합니다.


// Confirm the link is a sign-in with email link.
if (firebase.auth().isSignInWithEmailLink(window.location.href)) {
  // Additional state parameters can also be passed via URL.
  // This can be used to continue the user's intended action before triggering
  // the sign-in operation.
  // Get the email if available. This should be available if the user completes
  // the flow on the same device where they started it.
  var email = window.localStorage.getItem('emailForSignIn');
  if (!email) {
    // User opened the link on a different device. To prevent session fixation
    // attacks, ask the user to provide the associated email again. For example:
    email = window.prompt('Please provide your email for confirmation');
  }
  // The client SDK will parse the code from the link for you.
  firebase.auth().signInWithEmailLink(email, window.location.href)
    .then(function(result) {
      // Clear email from storage.
      window.localStorage.removeItem('emailForSignIn');
      // You can access the new user via result.user
      // Additional user info profile not available via:
      // result.additionalUserInfo.profile == null
      // You can check if the user is new or existing:
      // result.additionalUserInfo.isNewUser
    })
    .catch(function(error) {
      // Some error occurred, you can inspect the code: error.code
      // Common errors could be invalid email and invalid or expired OTPs.
    });
}

모바일 앱에서 로그인 완료
Firebase 인증에서는 Firebase 동적 링크를 사용하여 모바일 기기로 이메일 링크를 보냅니다. 모바일 애플리케이션을 통해 로그인을 완료하는 경우, 웹 과정에서처럼 애플리케이션에서 수신 애플리케이션 링크를 감지하고 기본 딥 링크를 파싱한 다음 로그인을 완료하도록 구성해야 합니다.

Android 애플리케이션에서 이메일 링크를 사용한 로그인 처리 방법을 자세히 알아보려면 Android 가이드를 참조하세요.

iOS 애플리케이션에서 이메일 링크를 사용한 로그인 처리 방법을 자세히 알아보려면 iOS 가이드를 참조하세요.

이메일 링크를 사용하여 연결/재인증
기존 사용자에게도 이 인증 방법을 연결할 수 있습니다. 예를 들어 전화번호 등 다른 제공업체로 전에 인증된 사용자의 경우 기존 사용자 계정에 이 로그인 방법을 추가할 수 있습니다.

이 경우 작업 뒷부분이 달라집니다.


// Construct the email link credential from the current URL.
var credential = firebase.auth.EmailAuthProvider.credentialWithLink(
    email, window.location.href);

// Link the credential to the current user.
firebase.auth().currentUser.linkWithCredential(credential)
  .then(function(usercred) {
    // The provider is now successfully linked.
    // The phone user can now sign in with their phone number or email.
  })
  .catch(function(error) {
    // Some error occurred.
  });

민감한 작업을 실행하기 전에 이메일 링크 사용자를 재인증하는 경우에도 사용할 수 있습니다.


// Construct the email link credential from the current URL.
var credential = firebase.auth.EmailAuthProvider.credentialWithLink(
    email, window.location.href);

// Re-authenticate the user with this credential.
firebase.auth().currentUser.reauthenticateWithCredential(credential)
  .then(function(usercred) {
    // The user is now successfully re-authenticated and can execute sensitive
    // operations.
  })
  .catch(function(error) {
    // Some error occurred.
  });

하지만 원래의 사용자가 로그인하지 않은 다른 기기에서 인증 과정이 종료되면 이 인증 과정이 완료되지 않을 수 있습니다. 이러한 경우 같은 기기에서 링크를 열도록 사용자에게 오류를 표시할 수 있습니다. 링크에 일부 상태를 전달하여 작업 유형 및 사용자 UID에 대한 정보를 표시할 수 있습니다.

이메일/비밀번호 로그인과 이메일 링크 로그인 구별하기
이메일을 사용한 비밀번호와 링크 기반 로그인을 모두 지원하는 경우 비밀번호/링크 사용자를 위한 로그인 방법을 구별하려면 fetchSignInMethodsForEmail을 사용합니다. 이렇게 하면 사용자에게 먼저 이메일 입력을 요청한 다음 로그인 방법을 제시하는 다음과 같은 식별자 우선 인증 과정에 유용합니다.

// After asking the user for their email.
var email = window.prompt('Please provide your email');
firebase.auth().fetchSignInMethodsForEmail(email)
  .then(function(signInMethods) {
    // This returns the same array as fetchProvidersForEmail but for email
    // provider identified by 'password' string, signInMethods would contain 2
    // different strings:
    // 'emailLink' if the user previously signed in with an email/link
    // 'password' if the user has a password.
    // A user could have both.
    if (signInMethods.indexOf(
            firebase.auth.EmailAuthProvider.EMAIL_PASSWORD_SIGN_IN_METHOD) != -1) {
      // User can sign in with email/password.
    }
     if (signInMethods.indexOf(
             firebase.auth.EmailAuthProvider.EMAIL_LINK_SIGN_IN_METHOD) != -1) {
       // User can sign in with email/link.
    }
  })
  .catch(function(error) {
    // Some error occurred, you can inspect the code: error.code
  });
}

위에 설명된 대로 이메일/비밀번호 및 이메일/링크는 다른 로그인 방법을 사용하지만 같은 firebase.auth.EmailAuthProvider(같은 PROVIDER_ID)로 간주됩니다.

다음 단계
사용자가 처음으로 로그인하면 신규 사용자 계정이 생성되고 사용자 인증 정보(사용자가 로그인할 때 사용한 사용자 이름과 비밀번호, 전화번호 또는 인증 제공업체 정보)에 연결됩니다. 이 신규 계정은 Firebase 프로젝트에 저장되며 사용자의 로그인 방법과 무관하게 프로젝트 내의 모든 앱에서 사용자 본인 확인에 사용할 수 있습니다.

앱에서 사용자의 인증 상태를 파악할 때 권장하는 방법은 Auth 객체에 관찰자를 설정하는 것입니다. 그러면 User 객체로부터 사용자의 기본 프로필 정보를 가져올 수 있습니다. 사용자 관리를 참조하세요.

Firebase 실시간 데이터베이스와 Cloud Storage 보안 규칙에서 로그인한 사용자의 고유 사용자 ID를 auth 변수로부터 가져온 후 이 ID를 통해 사용자가 액세스할 수 있는 데이터를 관리할 수 있습니다.

인증 제공업체의 사용자 인증 정보를 기존 사용자 계정에 연결하면 사용자가 여러 인증 제공업체를 통해 앱에 로그인할 수 있습니다.

사용자를 로그아웃시키려면 signOut을 호출합니다.

firebase.auth().signOut().then(function() {
  // Sign-out successful.
}).catch(function(error) {
  // An error happened.
});