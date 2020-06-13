# android-batch-resign-apk
安卓APK包批量重签名

### 解决安卓10跨应用获取ANDROID_ID不一致问题，经研究只要是同一个证书签名的应用就可以获取到相同的设备吗

```python
#!/usr/bin/python
# coding:UTF-8
import shutil
import os

storepass = 'xxx'
keypass = 'xxx'
keystore = '/data/python/xxx.keystore'
keyAlias = 'xxx'
origin_apkdir = '/data/upload'
tmp_dir = '/tmp'
filter_apk = ['xxx.apk']

# 列出指定目录的所有APK
src_apks = []

for file in os.listdir(origin_apkdir):
    if os.path.isfile(os.path.join(origin_apkdir, file)):
        extension = os.path.splitext(file)[1][1:]
        filename = os.path.basename(file)
        if filename in filter_apk:
            continue
        if extension in 'apk':
            src_apks.append(os.path.join(origin_apkdir, file))
print 'total apk count {}'.format(len(src_apks))

success_count = 0
try:
    for src_apk in src_apks:
        source = os.path.abspath(src_apk)
        target = os.path.join(tmp_dir, os.path.basename(source))

        # 复制源文件到临时目录
        print '====================================================================================='
        print 'copy {} => {}'.format(source, target)
        shutil.copy(source, target)

        print 'remove META-INF %s' % target
        os.popen('/usr/bin/zip -d %s META-INF/*' % target)

        basename_list = os.path.splitext(os.path.basename(source))
        output = os.path.join(tmp_dir, '{}_signed{}'.format(basename_list[0], basename_list[1]))

        # 拼装签名命令
        signer_str = '/usr/bin/jarsigner -verbose -storepass {} -keypass {} -keystore {} -signedjar {} -digestalg SHA1 -sigalg MD5withRSA {} {}'.format(storepass, keypass, keystore, output, target, keyAlias)
        print signer_str

        # 执行签名命令
        signer_result = os.popen(signer_str)
        if (signer_result.read() != ''):
            mv_cmd = '/bin/mv -f {} {}'.format(output, source)
            print mv_cmd
            os.popen(mv_cmd)

            rm_cmd = '/bin/rm -rf {}'.format(target)
            print rm_cmd
            os.popen(rm_cmd)
            print 'signer_result:\t', 'success'
            success_count += 1
        else:
            print 'signer_result:\t', 'fail result: ' + signer_result
except Exception, e:
    print 'Exception:\t', repr(e)

print '#=====================================================================================#'
print 'All done! success_count {}!  Any key to exit'.format(success_count)
print '#=====================================================================================#'
raw_input()
```
