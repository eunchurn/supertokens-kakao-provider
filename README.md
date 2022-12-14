# supertokens-kakao-provider

[![Publish Package to npmjs](https://github.com/eunchurn/supertokens-kakao-provider/actions/workflows/publish.yml/badge.svg)](https://github.com/eunchurn/supertokens-kakao-provider/actions/workflows/publish.yml) ![npm](https://img.shields.io/npm/dw/supertokens-kakao-provider) ![npm](https://img.shields.io/npm/v/supertokens-kakao-provider) [![Depfu](https://badges.depfu.com/badges/1d17edbcb6db21de2ce1aed5e57e988f/count.svg)](https://depfu.com/github/eunchurn/supertokens-kakao-provider?project_id=37165) ![npm bundle size](https://img.shields.io/bundlephobia/minzip/supertokens-kakao-provider) ![GitHub issues](https://img.shields.io/github/issues/eunchurn/supertokens-kakao-provider) ![NPM](https://img.shields.io/npm/l/supertokens-kakao-provider)

Kakao providers for SuperTokens

[SuperTokens](https://supertokens.com)애서 간편 로그인의 Kakao Custom provider를 제공합니다.

## Installation

```
npm install supertokens-kakao-provider
```

## Usage

### Express Backend: kakao SignIn and SignUp

```typescript
import supertokens from "supertokens-node";
import ThirdPartyEmailPassword from "supertokens-node/recipe/thirdpartyemailpassword";
import { Kakao } from "supertokens-kakao-provider";

superTokens.init({
  framework: "express",
  supertokens: {
    connectionURI: "http://your-auth-domain.com",
    apiKey: "your-secret-key",
  },
  appInfo: {
    appName: "your-auth-app-name",
    apiDomain: "https://your-api-domain.com",
    websiteDomain: "https://your-web-client-domain.com",
    apiBasePath: "/auth",
    websiteBasePath: "/auth",
  },
  recipeList: [
    ThirdPartyEmailPassword.init({
      providers: [
        kakao({
          clientId: process.env.KAKAO_ACCESS_KEY,
          clientSecret: process.env.KAKAO_ACCESS_SECRET,
          relativeRedirectUrl: "/auth/callback/kakao",
        }),
      ],
    }),
  ],
});
```

### Client

#### SignInAndUp click handler function

<https://supertokens.com/docs/thirdpartyemailpassword/custom-ui/thirdparty-login#step-1-redirecting-to-social--sso-provider>

```typescript
import ThirdPartyEmailPassword, {
  getAuthorisationURLWithQueryParamsAndSetState,
} from "supertokens-web-js/recipe/thirdpartyemailpassword";

enum ThirdPartyId {
  naver = "naver",
  kakao = "kakao", // use this
}

SuperTokens.init({
  appInfo: {
    appName: "your-auth-app-name",
    apiDomain: "https://your-api-domain.com",
    apiBasePath: "/auth",
  },
  recipeList: [ThirdPartyEmailPassword.init({}), Session.init()],
});

export const signInClicked = (thirdPartyId: ThirdPartyId) => async () => {
  try {
    const authUrl = await getAuthorisationURLWithQueryParamsAndSetState({
      providerId: thirdPartyId,
      authorisationURL: `${process.env.REACT_APP_HOST_URL}/auth/callback/${thirdPartyId}`,
    });
    window.location.assign(authUrl);
  } catch (err: any) {
    if (err.isSuperTokensGeneralError === true) {
      // this may be a custom error message sent from the API by you.
      window.alert(err.message);
    } else {
      window.alert("Oops! Something went wrong.");
    }
  }
};
```

#### Redirected page handler: Auth callback

<https://supertokens.com/docs/thirdpartyemailpassword/custom-ui/thirdparty-login#step-2-handling-the-auth-callback-on-your-frontend>

```typescript
import { thirdPartySignInAndUp } from "supertokens-web-js/recipe/thirdpartyemailpassword";

async function handleGoogleCallback() {
  try {
    const response = await thirdPartySignInAndUp();

    if (response.status === "OK") {
      console.log(response.user);
      if (response.createdNewUser) {
        // sign up successful
      } else {
        // sign in successful
      }
      window.location.assign("/home");
    } else {
      // SuperTokens requires that the third party provider
      // gives an email for the user. If that's not the case, sign up / in
      // will fail.

      // As a hack to solve this, you can override the backend functions to create a fake email for the user.

      window.alert(
        "No email provided by social login. Please use another form of login",
      );
      window.location.assign("/auth"); // redirect back to login page
    }
  } catch (err: any) {
    if (err.isSuperTokensGeneralError === true) {
      // this may be a custom error message sent from the API by you.
      window.alert(err.message);
    } else {
      window.alert("Oops! Something went wrong.");
    }
  }
}
```

## API

- `clientId`(required): `string` The client ID you received from Kakao API when you registered. 앱 REST API 키 [내 애플리케이션] > [앱 키]에서 확인 가능
- `clientSecret`(optional): `string` The client secret you received from Kakao API when you registered your application. if you set required, it required.
- `redirectUrl`(optional): `string` The URL to redirect to after the user has logged in. 인가 코드를 전달받을 서비스 서버의 URI [내 애플리케이션] > [카카오 로그인] > [Redirect URI]에서 등록
- `relativeRedirectUrl`(optional): `string` The relative URL to redirect to after the user has logged in. if you set `reidrectUrl`, this field not required.
