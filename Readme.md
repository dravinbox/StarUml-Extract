# StartUml破解流程

## 大致流程
```
#安装压缩工具
cnpm i -g asar

#拷贝一个副本，以防万一
cp app.asar app.asar.bak

#创建一个目录，准备放解压出来的源代码
mkdir code

#解压代码到code目录
asar extract app.asar code

#修改js代码,看下面部分

#删除文件
rm app.asar

#压缩代码，创建app.asar文件
asar pack code app.asar
```

## 修改代码
文件: \src\engine\license-manager.js

1. 修改validate函数,注释97行-100行，和第123行
2. 修改checkLicenseValidity函数

代码如下：
```js
/**
 * Check license validity
 *
 * @return {Promise}
 */
validate () {
  return new Promise((resolve, reject) => {
    try {
      // Local check
      var file = this.findLicense()
      if (!file) {
        reject('License key not found')
      } else {
        var data = fs.readFileSync(file, 'utf8')
        licenseInfo = JSON.parse(data)
        // 1. 注释掉
        /* if (licenseInfo.product !== packageJSON.config.product_id) {
          app.toast.error(`License key is for old version (${licenseInfo.product})`)
          reject(`License key is not for ${packageJSON.config.product_id}`)
        } else { */
          var base = SK + licenseInfo.name +
          SK + licenseInfo.product + '-' + licenseInfo.licenseType +
          SK + licenseInfo.quantity +
          SK + licenseInfo.timestamp + SK
          var _key = crypto.createHash('sha1').update(base).digest('hex').toUpperCase()
          if (_key !== licenseInfo.licenseKey) {
            reject('Invalid license key')
          } else {
            // Server check
            $.post(app.config.validation_url, {licenseKey: licenseInfo.licenseKey})
              .done(data => {
                resolve(data)
              })
              .fail(err => {
                if (err && err.status === 499) { /* License key not exists */
                  reject(err)
                } else {
                  // If server is not available, assume that license key is valid
                  resolve(licenseInfo)
                }
              })
          }
        // 2. 注释掉
        // }
      }
    } catch (err) {
      reject(err)
    }
  })
}

checkLicenseValidity () {
  this.validate().then(() => {
    setStatus(this, true)
  }, () => {
    // setStatus(this, false)
    // UnregisteredDialog.showDialog()
    // 3. 上面两行注释掉，修改为以下
    setStatus(this, true)
  })
}
```