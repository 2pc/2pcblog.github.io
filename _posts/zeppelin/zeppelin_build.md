编译，这里使用的zeppelin-0.8.0的zip包,如果直接使用0.8.0的all包注意jdk版本不能低于jdk1.8.0_144 ，反编译javax.ws是基于这个版本编译的

```
cd zeppelin-0.8.0
mvn clean  package  -Pbuild-distr  -DskipTests -Denforcer.skip=true -Dcheckstyle.skip=true -DskipRat=true
```
注意修改npm的源,也可以修改为ali的源

```
npm config set registry "http://registry.npmjs.org/"

```
