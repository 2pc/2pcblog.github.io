编译，这里使用的zeppelin-0.8.0的zip包

```
cd zeppelin-0.8.0
mvn clean  package  -Pbuild-distr  -DskipTests -Denforcer.skip=true -Dcheckstyle.skip=true -DskipRat=true
```
注意修改npm的源,也可以修改为ali的源

```
npm config set registry "http://registry.npmjs.org/"

```
