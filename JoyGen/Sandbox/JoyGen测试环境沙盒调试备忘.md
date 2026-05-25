* 当前对JoyGenProjectSandboxServiceImpl下的方法进行了projectType识别，当projectType为ei-java-spec-test时访问沙盒测试集群
* projectType判断逻辑集中在com.jd.fin.saas.joy.gen.wrapper.sandbox.DevCloudClientImpl#buildCommonHeaders(okhttp3.Request.Builder, java.lang.String)
* 由于存在大量线上环境的沙盒，其joygen_code_deploy_sandbo中的字段为test，当前该字段无法用于判断沙盒的实际环境