mecab-java-0.996
====
mecabのjavaバインディングをgradleで作成してairtifactoryする為のリポジトリ

<https://github.com/taku910/mecab/tree/master/mecab/java> からのfork

gradleで依存関係を書くだけでmecabを使いたくて作成。

cmecab-javaはbridjを使用してしており、bridjは自分でClassLoderしているのでelasticsearchの様に自前でClassLoaderしているようなものから使えない(´；ω；｀)

libMeCab.soはmecab-java-0.996-native-platform-linux-amd64.jarに内包されてます。
libMeCab.soのロードにjlibloader(<https://github.com/Boukefalos/jlibloader>)を使用してロードします。

## ビルド

```
gradle build
gradle uploadArchives
gradle uploadJni
```

## 使用方法

Gradleは以下のように記述

```java:build.gradle
repositories {
    maven { url 'https://animalmatsuzawa.github.io/mecab-java-0.996/'}
    maven { url 'https://github.com/Boukefalos/jlibloader/raw/mvn-repo/' }
}
dependencies {
  compile 'com.github.boukefalos:jlibloader:0.2'
  compile 'org.chasen.mecab:mecab-java-0.996:1.0'
  compile 'org.chasen.mecab:mecab-java-0.996-native-platform-linux-amd64:1.0'
}
```

javaバインディングのためのsoは以下のようにロード。

```java:
static {
 AccessController.doPrivileged((PrivilegedAction<Void>) () -> {
   try {
       Native.load("org.chasen.mecab", "MeCab");
     } catch (UnsatisfiedLinkError e) {
       throw new UnsatisfiedLinkError(
           "Cannot load the native code.\n"
               + "Make sure your LD_LIBRARY_PATH contains MeCab.so path.\n" + e);
     }
   return null;
  });
}
```

実際のlibMeCab.soはtempファイルとして展開される為、permitionの設定は、"loadLibrary.*"で記述必要。

```java:
grant {
  permission java.lang.RuntimePermission "loadLibrary.*";
};
```
