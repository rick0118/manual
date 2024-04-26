## Change Openbmc webui
We will use devtool and webui-vue to do it.

First,
```sh
git clone https://github.com/openbmc/webui-vue.git
```

### Init devtool workspace
cd /build/
```sh 
devtool create-workspace
```
after that we will have workspace /dir
List recipe and we can change
```sh
bitbake -s #Parsing all recipes
```
```sh
devtool modify webui-vue
```
cd to /source/
```sh
tree workspace/
```
if you want to chagne Logo cd to
```sh
sources/webui-vue/src/layouts #change LoginLayout.vue
```
After that we can rebuild
```sh
devtool build webui-vue
```
caus the code is zip, if we want to modify use patch!

```git
git commit  -m "[ADD]modify,Logo Change" sources/webui-vue/src/layouts/LoginLayout.vue
git commit  -m "[ADD]modify,Logo Change"  sources/webui-vue/src/assets/images/login-company-logo.svg

```
use devtool update-recipe and store in ur layer
```shbit
devtool update-recipe webui-vue -a /target dir/
```
if you use romulus, try to tree recipes-phosphor
```
/openbmc/meta-ibm/meta-romulus/recipes-phosphor
#can find webui-vue here
```