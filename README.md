# android-batch-resign-apk
安卓APK包批量重签名脚本(Linux版)

### 解决安卓10跨应用获取ANDROID_ID不一致问题

```python
#!/usr/bin/python
# coding:UTF-8
import shutil
import os
import sys
import logging

# logging setting
log = logging.getLogger()
log.setLevel(logging.DEBUG)
handler = logging.StreamHandler(sys.stdout)
handler.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
log.addHandler(handler)

# config
storepass = 'yoyou521'
keypass = 'yoyou521'
keystore = '/data/python/yoyou.keystore'
keyAlias = 'zcgame'
origin_apkdir = '/data/upload'
tmp_dir = '/tmp'
filter_apk = ['kepan.apk']

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

n = 0
try:
    for src_apk in src_apks:
        source = os.path.abspath(src_apk)
        target = os.path.join(tmp_dir, os.path.basename(source))

        # 复制源文件到临时目录
        log.info('=====================================================================================')
        log.info('copy {} => {}'.format(source, target))
        shutil.copy(source, target)

        log.info('remove META-INF %s' % target)
        os.popen('/usr/bin/zip -d %s META-INF/*' % target)

        basename_list = os.path.splitext(os.path.basename(source))
        output = os.path.join(tmp_dir, '{}_signed{}'.format(basename_list[0], basename_list[1]))

        # 拼装签名命令
        signer_str = '/usr/bin/jarsigner -verbose -storepass {} -keypass {} -keystore {} -signedjar {} -digestalg SHA1 -sigalg MD5withRSA {} {}'.format(storepass, keypass, keystore, output, target, keyAlias)
        log.info(signer_str)

        # 执行签名命令
        signer_result = os.popen(signer_str)
        if (signer_result.read() != ''):
            mv_cmd = '/bin/mv -f {} {}'.format(output, source)
            log.info(mv_cmd)
            os.popen(mv_cmd)

            rm_cmd = '/bin/rm -rf {}'.format(target)
            log.info(rm_cmd)
            os.popen(rm_cmd)
            log.info('signer_result:success')
            n += 1
        else:
            log.info('signer fail:{}'.format(signer_result))
except Exception, e:
    log.info('Exception:{}'.format(repr(e)))

log.info('#=====================================================================================#')
log.info('               All done! origin apk {}, signed apk {}.  Any key to exit'.format(len(src_apks), n))
log.info('#=====================================================================================#')
# raw_input()

```

#### window版本，主要是给运营人员使用，本地处理好之后在上传包

```python
#!/usr/bin/python
# coding:UTF-8
import shutil
import os

storepass = 'xxx'
keypass = 'xxx'
keystore = 'python/xxx.keystore'
keyAlias = 'xxx'
filter_apk = ['xxx.apk']

# 列出指定目录的所有APK
src_apks = []

origin_apkdir = os.getcwd() + '/origin/'
output_dir = os.getcwd() + '/signed/'
# 目录不存在则创建
if not os.path.exists(output_dir):
    os.mkdir(output_dir)
else:
    shutil.rmtree(output_dir, True)
    os.mkdir(output_dir)

for file in os.listdir(origin_apkdir):
    if os.path.isfile(os.path.join(origin_apkdir, file)):
        extension = os.path.splitext(file)[1][1:]
        filename = os.path.basename(file)
        if filename in filter_apk:
            continue
        if extension in 'apk':
            src_apks.append(os.path.join(origin_apkdir, file))


n = 0
try:
    for src_apk in src_apks:
        source = os.path.abspath(src_apk)
        basename = os.path.basename(source)
        target = os.path.join(output_dir, basename)

        print '======================================================================================='
        # 删除旧签名
        print 'remove META-INF %s' % source
        os.popen('winrar d -inul %s META-INF' % source)

        # 拼装签名命令
        signer_str = 'jarsigner -verbose -storepass {} -keypass {} -keystore {} -signedjar {} -digestalg SHA1 -sigalg MD5withRSA {} {}'.format(storepass, keypass, keystore, target, source, keyAlias)
        print signer_str

        # 执行签名命令
        signer_result = os.popen(signer_str)
        if (signer_result.read() != ''):
            print 'signer_result:\t', 'success'
            n += 1
        else:
            print 'signer_result:\t', 'fail result: ' + signer_result
except Exception, e:
    print 'Exception:\t', repr(e)

print '#=====================================================================================#'
print '               All done! origin apk {}, signed apk {}.  Any key to exit'.format(len(src_apks), n)
print '#=====================================================================================#'
raw_input()

```
