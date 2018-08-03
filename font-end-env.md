# 環境

目前後端使用的VisualStudio主要為了.net環境設計，而前端其實過去也有相當龐大的生態系。為了去利用這些資源，我們可以自行建構出一個適合我們自身需求的開發環境。

## 工具

目前用過的前端工具

### Node.js

Node.js提供了在伺服器上執行javascript的環境。衍生成開發階段時可執行各Node.js的模組，達到「編譯」前端檔案。

### npm

npm為Node.js預設提供的套件管理工具，可以拿來下載各種前端套件或開發套件。

### yarn
由於舊版的npm抓取套件慢，不同電腦抓取行為可能不同，yarn改善了這些問題。npm的指令大多都有yarn版

### webpack

Webpack是前端打包工具，並提供模組化開發方法。打包時可以壓縮、轉譯靜態文件。

### babel

Babel是JavaScript編譯器，可以將新的JavaScript語法編譯成ES5（IE10以上支援）

### JQuery

老牌的前端框架（或說函式庫），簡化JavaScript與HTML DOM的交互作用

### Vue.js

前端框架，可以MVVM的方法操作資料物件後，框架繪製視圖。有著漸進式開發導入容易的設計。

## 建立環境

目標是建立一個可以在IE10+\(9可能可以，8要做其他調整\)使用ES2015語法的 asp.net mvc 4專案，並且在編輯時自動將ES2015的js檔透過babel轉譯成ES5的js  
asp.net core 應該也是可行，因為前端環境/工具不相依後端環境/工具，但在自動轉譯的部分應該需要再調整？

### Node.js與npm環境確認

檢查是否有安裝Node.js，在"C:\Program Files\nodejs"路徑應該會有node.exe。如果沒有則可以到[Node.js官方網站]下載最新穩定安裝檔進行安裝。  
Node.js安裝完成後，檢查是否能執行npm指令。開啟命令提示字元（Win+R開啟執行，執行cmd），輸入以下指令

```
檢查npm版本
npm -version
5.6.0
```

如果出現「不是內部或外部命令、可執行的程式或批次檔。」則可能為環境變數未加入nodejs資料夾。  
以windows 10為例加入系統路徑變數：  
1. 搜尋「進階系統設定」執行  
2. 按「環境變數」按鈕  
3. 在「系統變數」區找到變數Path，按「編輯」  
4. 按「新增」按鈕，加入Node.js的安裝路徑，如"C:\Program Files\nodejs"

設定完環境變數後部分服務也許需要重新啟動才能獲得最新的系統環境變數，以命令提示字元而言，就是得關掉重新叫一個。

### 加入yarn到global
雖然npm在5.x版後也加入`package-lock.json`檔案解決取用套件版本不同問題，但yarn安裝套件速度還是快非常多，另外也支援取用bower套件，故還是建議使用yarn進行套件管理  
npm的`package-lock.json`與yarn的`yarn.lock`還是不太一樣，所以要用yarn還是npm進行套件管理可能得選一個

```
安裝到全域
npm install yarn -g
檢查安裝
yarn -version
```

#### 安裝到全域
npm 安裝到全域以Windows 7, 8 and 10 而言是安裝到`%AppData%\npm\node_modules`，在安裝nodejs時，應會在使用者的環境變數中加入`%AppData%\npm`，於是後續能執行套件指令，以`yarn`命令為例，就是執行`%AppData%\npm\yarn`

而yarn安裝到全域時是安裝到`%userprofile%\appdata\local\yarn`中，如果想透過yarn安裝全域套件執行，則需要將此路徑加入系統變數。

然而各個電腦的環境可能不一樣，目前認為儘量不在工作流程中使用全域套件，而是使用專案相依套件。

命令對照
```
安裝套件
npm install OOO
yarn add OOO
安裝開發用套件
npm install OOO --save--dev
yarn add OOO --dev
安裝套件到全域
npm install OOO -g
yarn global add OOO
```



### Asp.net mvc 4與Visual Studio開啟命令列捷徑

使用Visual Studio建立Asp.net mvc 4專案  
可以直接執行CMD並CD至「專案」資料夾後使用指令。\(不是到「方案」資料夾\)

```
C:\Users\drsmile>cd /d D:\Project\KASE_SBMS
D:\Project\KASE_SBMS\KASE_SBMS>
```

如果要簡化開啟命令列的過程，可以安裝[Open Command Line]Visual Studio擴充功能，即可直接在方案總管中對方案/專案/資料夾按右鍵開啟命令列並指到對應路徑。  
或者依據[Developer Command Prompt for Visual Studio]教學，新增外部工具開啟命令列（覺得沒有上面擴充功能好用）

### npm初始化，對專案資料夾加入各個前端套件

以下參考
[Using Vuejs in ASP .NET MVC Application]  
[Webpack 2 + Babel 6 + ES6 in ASP.NET Core 1.1 From Scratch]  
[Webpack: Creating dynamically named outputs for wildcarded entry files]  

#### 1. 建立package.json

使用指令建立package.json，途中會詢問專案的一些資訊。  
指令：

```
npm init
或
yarn init
```

範例：

    C:\Users\drsmile\Desktop\Test>npm init
    This utility will walk you through creating a package.json file.
    It only covers the most common items, and tries to guess sensible defaults.

    See `npm help json` for definitive documentation on these fields
    and exactly what they do.

    Use `npm install <pkg>` afterwards to install a package and
    save it as a dependency in the package.json file.

    Press ^C at any time to quit.
    package name: (test)
    version: (1.0.0)
    description:
    entry point: (index.js)
    test command:
    git repository:
    keywords:
    author:
    license: (ISC)
    About to write to C:\Users\drsmile\Desktop\Test\package.json:

    {
      "name": "test",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC"
    }


    Is this ok? (yes)

    C:\Users\drsmile\Desktop\Test>

此package.json檔會記錄這個「專案」資料夾參考的npm模組，如同NuGet的packages.config會記錄專案所參考的NuGet套件

#### 2. 加入webpack開發階段相依

將使用webpadck協助進行JavaScript的打包，透過指令安裝

指令：

```
npm install --save-dev webpack
npm install --save-dev webpack-cli
或
yarn add webpack --dev
yarn add webpack-cli --dev
```

#### 3. 加入babel相依

使用指令加入babel相關套件：

```
npm install babel-core babel-loader babel-preset-env --save-dev
npm install babel-polyfill --save
```

`babel-core` 為核心功能  
`babel-loader` 供webpack使用的loader  
`babel-preset-env`表示依據環境設定轉換js語法，如果環境沒特別設定，其功能與`babel-preset-latest`類似，支援所有ECMA版本與stage4特性  
目前猜測是轉換到ES5版本  
[[译]babel-preset-env]()

```javascript
// 原始
input.map(item => item + 1);
// 轉換
input.map(function (item) {
    return item + 1;
});
```

`babel-polyfill`提供在舊瀏覽器上執行新規範的JavaScript API，譬如Array.find

#### 4. 設定webpack.config.js

webpack依據專案下的webpack.config.js進行打包，在專案資料夾下新增webpack.config.js檔後新增webpack設定，以[Webpack 2 + Babel 6 + ES6 in ASP.NET Core 1.1 From Scratch]的範例說明

```javascript
var path = require("path");
module.exports = {
    entry: [
        "babel-polyfill",
        "./Scripts/main"
    ],
    output: {
        publicPath: "/js/",
        path: path.join(__dirname, "/wwwroot/js/"),
        filename: "main.build.js"
    },
    module: {
        rules: [{ //在webpack3這裡是loaders,wepack4改用rules
            exclude: /node_modules/,
            loader: "babel-loader",
            query: {
                presets: ["env"] //babel後續推薦使用env
            }
        }]
    },
};
```

其中`entry`是一個陣列的時候，表示將這些路徑的js封裝成單一js，適合SPA\(\(Single Page Application\)設計；當寫成物件時，則每個屬性下的陣列封裝一個js，譬如：

```javaScript
entry:{
    AAA:["AAA1.js","AAA2.js"],
    BBB:["BBB1.js","BBB2.js"],
}
```

`output`指定封裝的js要輸出的位置，`fileName`可以是`"[name].js"`，則前面多個封裝的js會分開包裝。  
`module`部分設定webpack使用Babel進行轉譯

---

為了符合原本MPA\(Multi-page Application\)設計，在各個頁面載入自己需要的.js檔案，參考[Webpack: Creating dynamically named outputs for wildcarded entry files]修改webpack.config.js設計如下：

```javascript
var path = require("path");
const glob = require("glob");
const entryArray = glob.sync("src/**/**.js", { matchBase: true });
const entryObject = entryArray.reduce((acc, item) => {
    const name = item.replace(".js", "").replace("src/","");
    acc[name] = `./${item}`;
    return acc;
}, {});
entryObject["babel-polyfill"] = "babel-polyfill";
module.exports = {
    entry: entryObject,
    output: {
        path: path.join(__dirname, "/dist/"),
        filename: "[name].js"
    },
    devtool: "source-map",
    module: {
        rules: [{
            exclude: /node_modules/,
            loader: "babel-loader",
            query: {
                presets: ["env"]
            }
        }]
    }
};
```

先安裝開發階段[node-glob]

```
npm install -save-dev glob
```

node-glob提供查找資料夾檔案的功能。這裡將開發的.js存放在`src`，打包的.js存放在`dist`，在`module.exports`前面的Code會查詢`src`的所有資料夾下的所有js，組成entryObject物件，使得後續webpack打包可以依原檔案路徑分別打包。

#### 5. 執行webpack打包

在設定webpack.config.js後，即可用webpack指令進行打包  
必須有在全域上安裝webpack-cli，才有webpack指令能用，否則看下一段package.json的指令

指令：

```
開發版
webpack --mode development
上線版
webpack --mode production
```

範例：

```
PS C:\Users\drsmile\source\repos\WebApplication1\WebApplication1> webpack
[ 'Scripts/beforeBuild/AAA.js',
  'Scripts/beforeBuild/BBB.js',
  'Scripts/beforeBuild/Home/Index.js' ]
Hash: 9a58f3871f3a5bea70c8
Version: webpack 3.11.0
Time: 1251ms
                Asset     Size  Chunks                    Chunk Names
    babel-polyfill.js   264 kB       0  [emitted]  [big]  babel-polyfill
        Home/Index.js  2.61 kB       1  [emitted]         Home/Index
               BBB.js  2.59 kB       2  [emitted]         BBB
               AAA.js  2.59 kB       3  [emitted]         AAA
babel-polyfill.js.map   358 kB       0  [emitted]         babel-polyfill
    Home/Index.js.map  2.74 kB       1  [emitted]         Home/Index
           BBB.js.map  2.69 kB       2  [emitted]         BBB
           AAA.js.map  2.69 kB       3  [emitted]         AAA
  [90] (webpack)/buildin/global.js 509 bytes {0} [built]
 [125] ./Scripts/beforeBuild/AAA.js 57 bytes {3} [built]
 [126] ./Scripts/beforeBuild/BBB.js 57 bytes {2} [built]
 [127] ./Scripts/beforeBuild/Home/Index.js 71 bytes {1} [built]
    + 326 hidden modules
PS C:\Users\drsmile\source\repos\WebApplication1\WebApplication1>
```

可看到webpack將Scripts/beforeBuild/下各js重新打包到./Scripts/beforeBuild/

#### 6. webpack監看自動打包

如果每次編輯js都要執行webpack打包，對於開發流程就不夠便利。webpack有[Watch][webpack Watch]命令，監看檔案異動時進行打包。  
```
webpack --mode development -w
```
目前推測，寫在package.json scripts下的指令會採用專案的相依模組執行，可以在此寫對應的開發用命令  

package.json:
```javascript 
{
    ...
    "scripts":{
        "dev": "webpack --mode development -w",
        "build": "webpack --mode production"
    }
    ...
}
```

```
npm run dev
或
yarn run dev
```

##### WebPack Task Runner
WebPack Task Runner 暫時不支援webpack 4   
同時可以安裝[WebPack Task Runner]Visual Studio擴充套件，在專案開啟時就執行Watch指令。在安裝此套件後，開啟工作執行器總管，對webpack.config.js&gt;Watch&gt;Watch-Development右鍵&gt;繫結&gt;專案開啟。

WebPack Task Runner如果發生類似以下的nodejs執行錯誤，可能原因是VisualStudio所參考的nodejs版本較舊，可參考[update-node-version-in-visual-studio-2017]將Visual Studio的外部web開發工具參考目錄的順序以系統變數為優先（或直接指定到最新的nodejs目錄）
```
let webpackCliInstalled = false;
^^^
SyntaxError: Block-scoped declarations (let, const, function, class) not yet supported outside strict mode
```

[Node.js官方網站]:https://nodejs.org/en/
[Open Command Line]:https://marketplace.visualstudio.com/items?itemName=MadsKristensen.OpenCommandLine
[Developer Command Prompt for Visual Studio]:https://docs.microsoft.com/en-us/dotnet/framework/tools/developer-command-prompt-for-vs
[Using Vuejs in ASP .NET MVC Application]:http://queryasp.net/39/using-vuejs-in-asp-net-mvc-application
[Webpack 2 + Babel 6 + ES6 in ASP.NET Core 1.1 From Scratch]:https://medium.com/@vinjenks/webpack-2-babel-6-es6-in-asp-net-core-1-1-from-scratch-ed7a6a289ba7
[Webpack: Creating dynamically named outputs for wildcarded entry files]:https://hackernoon.com/webpack-creating-dynamically-named-outputs-for-wildcarded-entry-files-9241f596b065
[babel套裝設定]:https://babeljs.io/docs/plugins/preset-env
[node-glob]:https://github.com/isaacs/node-glob
[webpack Watch]:https://webpack.js.org/api/cli/#watch-options  
[WebPack Task Runner]:https://marketplace.visualstudio.com/items?itemName=MadsKristensen.WebPackTaskRunner
[update-node-version-in-visual-studio-2017]:https://stackoverflow.com/questions/43849585/update-node-version-in-visual-studio-2017
[[译]babel-preset-env]:https://icyfish.me/2017/05/18/babel-preset-env/