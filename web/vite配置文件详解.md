```js
import { defineConfig } from "vite";
import vue from '@vitejs/plugin-vue'
const path = require('path')
function pathResolve(dir: String) {
  return path.join(__dirname, dir);
}
export default defineConfig({
  root: "",//表示项目跟目录
  /**端口号 可改 */
  base: './',
  mode: '',//设置开发模式还是生产模式
  plugins: [vue()],
  resolve: {
    alias: [//别名配置
      // 键必须以斜线开始和结束
      {
        find: /\/@\//,
        replacement: pathResolve('src') + '/',
      }, {
        find: /\/@assets\//,
        replacement: pathResolve('src/assets') + '/',
      }, {
        find: /\/comp\//,
        replacement: pathResolve('../../../src/components') + '/',
      }, {
        find: /\/@router\//,
        replacement: pathResolve('src/router') + '/',
      }, {
        find: /\/@views\//,
        replacement: pathResolve('src/views') + '/',
      }, {
        find: /\/@vuex\//,
        replacement: pathResolve('src/vuex') + '/',
      }, {
        find: /\/@mock\//,
        replacement: pathResolve('src/mock') + '/',
      },
    ],
    //why remove it , look for https://github.com/vitejs/vite/issues/6026
    // extensions: ['.js', '.ts', '.jsx', '.tsx', '.json', '.vue', '.mjs']
  },// 跨域设置
  css: {
    modules: {},//配置css modules的行为
    postcss: {},
    preprocessorOptions: {//指定传递给css预处理器的选项
      sass: {
        additionalData: `$injectedColor: orange;`
      }
    }
  },//关于css在项目中的相关配置
  // open: true,

  server: {
    host: "127.0.0.1",//指定服务器主机名
    port: 80,//指定服务器端口。注意：如果端口已经被使用，Vite 会自动尝试下一个可用的端口，所以这可能不是服务器最终监听的实际端口
    strictPort: true,//配合port,如果端口被占用 设置为true，就会自动退出不会尝试寻找下一个出口
    proxy: {//关于代理的配置w
      // '/api': {
      //   target: 'https://blog.csdn.net/weixin_45292658',//表示代理的目标地址
      //   changeOrigin: true,
      //   rewrite: path => path.replace(/^\/api/, '')
      // }
    },
  },
  preview: {
    port: 5003,
    host: "0.0.0.0",
    strictPort: true
  },
  // 忽略导入文件类型的扩展名列表
  // build打包构建的时候需要进行的相关配置
  build: {
    target: "",//关于配置浏览器兼容性，转换过程将会由esbuild执行，如果是自定义目标也可以试一个es版本、一个浏览器版本或者是多个数组[]
    outDir: "",//打包输出路径   默认为 dist
    assetsDir: "",//指定生成静态资源的存放路径
    assetsInlineLimit: 4096,//kb,小于此阈值的导入或引用资源将内联为base64编码，以避免额外的http请求。设置为0 可以完全禁用此项。
    cssCodeSplit: true,//启用/禁用 CSS 代码拆分。当启用时，在异步 chunk 中导入的 CSS 将内联到异步 chunk 本身，并在块加载时插入。
    // 如果禁用，整个项目中的所有 CSS 将被提取到一个 CSS 文件中 
    sourcemap: true,//默认为false;构建后是否生成source map文件
    rollupOptions: {//自定义底层的RollUp打包配置，这与Rollup配置文件导出的选项相同，并将与Vite的内部Rollup选项合并

    },
    emptyOutDir: true,//boolean 如果构建输出目录在根目录的话，会抛出一个警告避免删除掉重要文件。
    commonjsOptions: {},//传递给@rollup/plugin-commonjs 插件的选项。
    brotliSize: false,//默认为true 弃用/禁用压缩大小报告，压缩大型输出文件可能会很慢，所以需要禁用该功能从而提高大型项目的构建功能
    chunkSizeWarningLimit: 500,//kb chunk大小警告的限制 
    terserOptions: {//相关配置  https://terser.org/docs/api-reference#minify-options
      parse: {},//如果希望指定一些其他 解析选项 则传递一个对象
      compress: {//默认为空对象跳过压缩，或者传递自定义压缩选项
        drop_console: true,//移除console.log
        drop_debugger: true
      }
    }
  },
  //引入第三方配置
  optimizeDeps: {
    include: ["element-plus/lib/locale/lang/zh-cn"],
  },
})
```

