작업 중 여러 Repository에서 같은 코드를 사용하게 되었다. 예를 들면 아래와 같은 코드이다.
```typescript
/**
 * Smart Contract 메서드를 정의한 클래스
 */
class SmartContract {
	// ...
}
```

`SmartContract` 클래스는 말 그대로 어떤 Smart Contract의 메서드를 정의하여 Call 하는 인터페이스의 역할을 담당한다.
문제는 Node.js 환경의 여러 프로젝트에서 해당 코드 복사하여 사용하고 있는데, 해당 코드에서 버그가 발견되면 고치는데 공수가 많이 들어, 이를 Package로 만들어 보자는 아이디어가 떠올랐다.

# Github Packages
모든 프로젝트는 Github에서 관리하기 때문에, Registry는 NPM이 아닌 Github Packages를 사용하기로 한다. 패키지를 만드는 방법은 다음과 같다.
1. 패키지 프로젝트 및 Github 저장소 생성
2. Github Personal Access Token 생성
3. 프로젝트 설정
4. 배포

## 패키지 프로젝트 생성
다음과 같이 생성을 진행한다.
```shell
$ mkdir smart-contract && cd smart-contract
$ yarn init
yarn init v1.22.19
question name (testdir): @{username}/{repo_name}
question version (1.0.0): 1.0.0
question description:
question entry point (index.js): dist/index.js
question git repository: https://github.com/{username}/{repo_name}.git
question author:
question license (MIT):
question private:
success Saved package.json
✨  Done in 87.70s.
```

`pakcage.json` 파을 구조는 다음과 같다. `{username}`과 `{repo_name}`만 잘 구별하여 작성하면 된다.
```json
{
	"name": "@{username}/{repo_name}",
	"version": "1.0.0",
	"main": "dist/index.js",
	"types": "dist/index.d.ts",
	"repository": "https://github.com/{username}/{repo_name}.git",
	"publishConfig": {
		"registry": "https://npm.pkg.github.com"
	},
	"author": "boy672820",
	"license": "MIT",
	"scripts": {
		"build": "tsc"
	},
	"devDependencies": {
		"typescript": "^5.0.4"
	},
	"files": [
		"dist",
		"LICENSE",
		"README.md",
		"package.json"
	],
	"dependencies": {
		"ethers": "^6.3.0"
	}
}
```

### .npmrc 파일 설정
npm, yarn 과 같은 패키지 매니저가 특정 패키지를 설치할 때 참고하는 파일이다. 우리가 생성할 패키지의 Registry는 NPM이 아닌 Github이기 때문에 패키지 매니저에게 이를 알려주어야 한다.

홈 디렉토리에 `.npmrc` 파일을 생성한 후 다음과 같이 설정한다.
```shell
$ echo "registry=https://registry.yarnpkg.com/\n\n@{username}:registry=https://npm.pkg.github.com\n//npm.pkg.github.com/:_authToken={github_token}\nalways-auth=true" >> ~/.npmrc
```

파일은 다음과 같다.
```
registry=https://registry.yarnpkg.com/

@{username}:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken={github_token}
always-auth=true
```
`{github_token}`은 [개인용 엑세스 토큰](https://docs.github.com/ko/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)으로 우리가 만든 패키지를 등록하고 접근할 수 있게 해준다.

## Github Personal Access Token
우리가 `.npmrc`파일에서 지정한 `{github_token}`을 생성해 주어야한다.
프로필 이미지에서 `Settings > Developer settings > Personal Access Tokens > Tokens (classic)` 으로 접속하여 키를 생성한다.

**생성 시 `write:packages`를 필수로 선택해야 한다.**

![github_settings.png](https://boy672820.github.io/assets/images/2023-05-20-npm-package/github_settings.png)
![github_profile_developers.png](https://boy672820.github.io/assets/images/2023-05-20-npm-package/github_profile_developers.png)
![github_generate_token_package.png](https://boy672820.github.io/assets/images/2023-05-20-npm-package/github_generate_token_package.png)

## 배포
배포 준비가 완료되었다. 아래 명령어로 간단하게 배포가 가능하다.

```shell
$ yarn publish
...
```

Github Actions를 활용하면 편하다. `.github/workflows/publish.yml` 파일을 아래와 같이 생성해 주자

```yml
name: publish

on:
	release:
		types: [created]

jobs:
	publish-github-registry:
		runs-on: ubuntu-latest
		steps:
			- uses: actions/checkout@v3
			- uses: actions/setup-node@v3
			with:
				node-version: {your_node_version}
				registry-url: https://npm.pkg.github.com/
			- run: yarn install
			- run: yarn build
			- run: yarn publish
			env:
				NODE_AUTH_TOKEN: ${{secrets.GHPR_TOKEN}}
```
