<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
    <!-- 使用pom.xml中配置的仓库 -->
<!--    <mirrors>-->
<!--        <mirror>-->
<!--            <id>central-mirror</id>-->
<!--            <name>Central Repository Mirror</name>-->
<!--            <url>https://repo.maven.apache.org/maven2</url>-->
<!--            <mirrorOf>central</mirrorOf>-->
<!--        </mirror>-->
<!--    </mirrors>-->
    
    <profiles>
        <profile>
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <repositories>
                <repository>
                    <id>central</id>
                    <name>libs-releases</name>
                    <url>https://artifactory.jd.com/libs-releases</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>snapshots</id>
                    <name>libs-snapshots</name>
                    <url>https://artifactory.jd.com/libs-snapshots</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                </repository>
                <repository>
                    <id>maven-central</id>
                    <name>Maven Central</name>
                    <url>https://repo.maven.apache.org/maven2</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <name>plugins-releases</name>
                    <url>https://artifactory.jd.com/plugins-releases</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>snapshots</id>
                    <name>plugins-snapshots</name>
                    <url>https://artifactory.jd.com/plugins-snapshots</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                </pluginRepository>
                <pluginRepository>
                    <id>maven-central</id>
                    <name>Maven Central</name>
                    <url>https://repo.maven.apache.org/maven2</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
</settings>