---
title: gradle 问题总结
date: 2016-10-30 11:47:21
tags:

---

<font size=4>好久没更新博客了，因此把最近遇到的问题记录下来，主要包括以下2个问题，以供大家参考！</font>

<font size=3>1. Android studio升级到2.2.0之后，Robolectric配置的单元测试无结果输出</font>

在Android studio更新到2.2.0之后，发现单元测试用例都跑不通，苦思不得其解

更新前的配置：

	Android studio 2.1.9
	Gradle plugin version: 2.2.0
	Gradle version: 2.14.1
	Build tools version: 24.0.2
	JUnit 4.12
	Robolectric 3.1.2
	Mockito 1.10.19

更新后的配置

	Android studio 2.1.9
	Gradle plugin version: 2.2.0
	Gradle version: 2.14.1
	Build tools version: 24.0.2
	JUnit 4.12
	Robolectric 3.1.2
	Mockito 1.10.19

仅仅是Android studio升级了，其他都没有变化，结果单元测试却跑不过。
首先从兼容性入手，排查是否是因为Android studio 2.2.0与Robolectri、JUnit、Mockito的版本不兼容问题，几经尝试，排除了兼容性问题。最后发现Robolectric每次运行的时候会去读取工程的Manifest文件,默认路径如下：

	if (FileFsFile.from(buildOutputDir, "manifests").exists()) {
      manifest = FileFsFile.from(buildOutputDir, "manifests", "full", flavor, abiSplit, type, DEFAULT_MANIFEST_NAME);
    } else {
      manifest = FileFsFile.from(buildOutputDir, "bundles", flavor, abiSplit, type, DEFAULT_MANIFEST_NAME);
    }	
在我的工程里面先前是：

	build/intermediates/manifests/full/debug/AndroidManifest.xml
在Android studio升级后，Manifest路径变成了：

	build/intermediates/manifests/aapt/debug/AndroidManifest.xml
因此单元测试用例都无法通过，因此需要修改Robolectric读取的Manifest默认路径，我通过反射方法修改了其默认路径：

		@Override
		protected AndroidManifest getAppManifest(final Config config) {
		AndroidManifest appManifest = super.getAppManifest(config);
		if (config.manifest() != null && !"".equals(config.manifest())) {
		    FileFsFile manifestFile = FileFsFile.from(config.manifest());
		    Class<? extends AndroidManifest> appManifestClass = null;
		    try {
		        appManifestClass = appManifest.getClass();
		        Field androidManifestFileField;
		        try {
		            androidManifestFileField = appManifestClass.getDeclaredField("androidManifestFile");
		        } catch (NoSuchFieldException e) {
		            androidManifestFileField = appManifestClass.getSuperclass().getDeclaredField("androidManifestFile");
		        }
		        androidManifestFileField.setAccessible(true);
		        androidManifestFileField.set(appManifest, manifestFile);
		    } catch (NoSuchFieldException e) {
		        e.printStackTrace();
		    } catch (IllegalAccessException e) {
		        e.printStackTrace();
		    }
		}
		return appManifest;
		}

最后在配置的Config的时候直接指定Manifest的路径即可：

	@Config(constants = BuildConfig.class,
        manifest = "src/main/AndroidManifest.xml",
        shadows = {ShadowMyManager.class,}, sdk = 23)
因此该问题就解决了！


<font size=3>2. gradle利用maven-publish脚本打包library到maven仓库问题</font>
由于项目需要，需要将自己的library打包上传到公司的maven服务器，由于以前用的是maven插件来打包library库，但是发现十分麻烦，而且不太灵活；最后发现有maven-publish插件，于是就通过maven-publish来写了个脚本，实现一键打包上传到仓库，十分方便，而且通用、灵活。

既可以上传到私有maven，又可以上传到公司的maven仓库，debug和release版本可以同时发布。

	apply plugin: 'maven-publish'
	def getRepositoryUsername() {
    return hasProperty('NEXUS_USERNAME') ? NEXUS_USERNAME : ""
	}

	def getRepositoryPassword() {
    return hasProperty('NEXUS_PASSWORD') ? NEXUS_PASSWORD : ""
	}

	task androidSourcesJar(type: Jar) {
    classifier = 'sources'
    from android.sourceSets.main.java.srcDirs
	}
	
	publishing {
	    repositories {
	        maven {
	            credentials {
	                username getRepositoryUsername()
	                password getRepositoryPassword()
	            }
	            name 'Repository-Remote-Release'
	            url RELEASE_REPOSITORY_URL
	        }
	        maven {
	            credentials {
	                username getRepositoryUsername()
	                password getRepositoryPassword()
	            }
	            name 'Repository-Remote-Snapshot'
	            url SANPSHOT_REPOSITORY_URL
	        }
	    }
    publications {

	        // Create different publications for every build types (debug and release)
	        android.libraryVariants.all { variant ->
	            "${variant.name}"(MavenPublication) {
	                //Create aar or jar
	                if (("aar").equals(POM_PACKAGING)) {
	                    artifact bundleRelease
	                } else if (("jar").equals(POM_PACKAGING)) {
	                    def name = variant.buildType.name
	                    def task = project.tasks.create "jar${name.capitalize()}", Jar
	                    task.dependsOn variant.javaCompile
	                    task.from variant.javaCompile.destinationDir
	                    task.exclude("**/R.class")
	                    task.exclude("**/R\$**.class")
	                    artifact task
	                }
	                artifact androidSourcesJar
	
	                if ("${variant.name}".equalsIgnoreCase("debug")) {
	                    publications."${variant.name}".version = project.version + "-SNAPSHOT"
	
	                } else {
	                    publications."${variant.name}".version = project.version
	                }
	                artifactId = POM_ARTIFACT_ID
	                groupId = GROUP
	
	                pom.withXml {
	                    def dependenciesNode = asNode().appendNode('dependencies')
	
	                    def configurationNames = ["${variant.name}Compile", 'compile']
	
	                    configurationNames.each { configurationName ->
	                        this.configurations[configurationName].allDependencies.each {
	                            if (it.group != null && it.name != null) {
	                                def dependencyNode = dependenciesNode.appendNode('dependency')
	                                dependencyNode.appendNode('groupId', it.group)
	                                dependencyNode.appendNode('artifactId', it.name)
	                                dependencyNode.appendNode('version', it.version)
	
	                                if (it.excludeRules.size() > 0) {
	                                    def exclusionsNode = dependencyNode.appendNode('exclusions')
	                                    it.excludeRules.each { rule ->
	                                        def exclusionNode = exclusionsNode.appendNode('exclusion')
	                                        exclusionNode.appendNode('groupId', rule.group)
	                                        exclusionNode.appendNode('artifactId', rule.module)
	                                    }
	                                }
	                            }
	                        }
	                    }
	                }
	            }
	        }


    }
	}
	task publishRelease(dependsOn: [
	        'generatePomFileForReleasePublication',
	        'publishReleasePublicationToRepository-Remote-ReleaseRepository'
	])
	task publishDebug(dependsOn: [
	        'generatePomFileForDebugPublication',
	        'publishDebugPublicationToRepository-Remote-SnapshotRepository'
	])
	task publish(overwrite: true, dependsOn: [
	        'publishRelease',
	        'publishDebug'
	])

	/**
	 *
	 * 执行gradle publish命令时,默认执行以下task,忽略其他task
	 * publishDebugPublicationToRepository-Remote-SnapshotRepository
	 * publishReleasePublicationToRepository-Remote-ReleaseRepository
	 *
	 */
	gradle.taskGraph.whenReady { taskGraph ->
	    def tasks = taskGraph.getAllTasks()
	    if (tasks.find { ("publish").equals(it.name) }) {
	        tasks.findAll {
	                    ("publishDebugPublicationToRepository-Remote-ReleaseRepository").equals(it.name) ||
	                    ("publishReleasePublicationToRepository-Remote-SnapshotRepository").equals(it.name)
	        }.each { task ->
	            task.enabled = false
	            task.group = null
	            return false
	        }
	    }
	}
私有的Maven可以自己配置，不在赘述。
在需要发布library的模块中的build.gradle脚本中直接引用该脚本即可：

	apply from: "$rootDir/maven_publish.gradle"
执行命令：

	gradle publish（分别发布release和debug版本到release和snapshot仓库中）
	gradle publishDebug（发布debug版本到snapshot仓库中）
	gradle publishRelease（发布release版本到release仓库中）

暂定写到这儿，还有几个问题，下次再总结！！！