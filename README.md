# 컴포넌트 라이브러리 npm 배포 과정(vite,pnpm)

## vite로 react 프로젝트 생성
`pnpm create vite . --template react-ts` [참고](https://mycodings.fly.dev/blog/2022-11-19-using-vite-rather-than-create-react-app-cra#google_vignette)


## storybook 설치
root 파일에서 터미널에 접근후 아래 명령어 입력
`npx storybook@latest init`

## 실제 배포하기
호스팅된 url을 package.json에 적어줘야하기 때문에 먼저 배포.  
배포하는 방법까지 설명하지 않음. 나는 vercel 이용함  
https://storybook-npm-publish.vercel.app

## 빌드 설정

### package.json
어떻게 npm에 등록하고 외부 프로젝트에 어떻게 내보낼지 작성해준다.

아래는 예시 코드이다.

```json
{
	// name: 프로젝트의 이름. 패키지의 고유 식별자이다
  "name": "storybook-npm-publish",
	// npm publish 할때마다 최소 patch는 올려주어야함
  "version": "0.0.1",
	// 프로젝트 단위에서 es 모듈을 사용할 수 있게 함으로써 import를 사용할 수 있게 해준다. 그냥 type: module로 설정하면 됨
  "type": "module",
	// 패키지의 주요 진입점 파일
  "main": "./dist/index.js",
	// 패키지의 TypeScript 타입 정의 파일
  "types": "./dist/index.d.ts",
	// 모듈을 외부에 공개할 때 사용하는 설정.
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "require": "./dist/index.umd.cjs"
    },
    "./style.css": "./dist/style.css"
  },
	// 패키지에 포함되어야 하는 파일과 디렉토리를 나타낸다. dist 디렉토리 내의 모든 파일이 포함된다.
  "files": [
    "dist"
  ],
  "repository": "https://github.com/ryu9663/storybook-npm-publish.git",
  "author": "Junyeol Ryu",
  "homepage": "https://storybook-npm-publish.vercel.app/",
	
	"scripts": {
    "start": "vite",
    "build:lib": "tsc && vite build",
    "build:storybook": "storybook build -o dist && touch ./dist/.nojekyll",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview",
    "dev": "storybook dev -p 6006"
  },
```

### vite.config.ts
어떻게 파일을 말아줄지



```typescript
import { defineConfig } from "vite";
import * as path from "path";
import dts from "vite-plugin-dts";
import tsconfigPaths from "vite-tsconfig-paths";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [dts(), tsconfigPaths()],

  server: {
    port: 3000,
  },

  build: {
		// 빌드할 라이브러리에 대한 설정
    lib: {
			// 라이브러리의 진입점
      entry: path.resolve(__dirname, "src/index.tsx"),
			// 라이브러리 이름
      name: "index",
      fileName: "index",
    },
    rollupOptions: {
			// 번들에 포함시키지 않을 외부 종속성
      external: ["react"],
			// 번들의 출력 옵션 설정
      output: {
        globals: {
          react: "React",
        },
      },
    },
		// CommonJS 번들러에 대한 옵션을 정의한다.
    commonjsOptions: {
      esmExternals: ["react"],
    },
  },

	// 절대경로
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});

```
### tsconfig.json
절대경로를 제외하고는 create vite에서 만들어진 tsconfig.json에서 바꾸지 않음.
자세히 알고 싶으면 [이 게시글](https://junghyeonsu.com/posts/deploy-simple-util-npm-library/#%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%84%A4%EC%A0%95) 참고

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}

```

### npm publish
[이 게시글](https://www.daleseo.com/js-npm-publish/#google_vignette) 보고 따라하면 됨

