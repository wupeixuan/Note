报错
```
2019-09-24 19:43:05,315 - ERROR #[operation-plataform,,,]# [http-nio-8111-exec-3] c.g.c.j.JDBCUtil [JDBCUtil.java:173]: SQLException: {}
java.sql.SQLException: [Simba][ImpalaJDBCDriver](500352) Error getting the parameter data type: HIVE_PARAMETER_QUERY_DATA_TYPE_ERR_NON_SUPPORT_DATA_TYPE
```


接受返回结果的dto
```
public class ProjectLivenessDTO {

	private String projectid;

	private Long livenessDateCount;

	public String getProjectid() {
		return projectid;
	}

	public void setProjectid(String projectid) {
		this.projectid = projectid;
	}

	public Long getLivenessDateCount() {
		return livenessDateCount;
	}

	public void setLivenessDateCount(Long livenessDateCount) {
		this.livenessDateCount = livenessDateCount;
	}
```


implala 数据库中字段为 datetime 时，参数要为 date 类型，而不能为 string 类型，并且查询出来的字段名需要和 dto 中的属性对应（livenessDateCount->liveness_date_count）。

```
sqlBuilder.append("SELECT projectid, count(*) AS liveness_date_count FROM ( SELECT comb.projectid, comb.mday FROM project_liveness_combine comb LEFT JOIN product_project_re re ON ( re.project_id = comb.projectid AND re.product_code = comb.product_code ) WHERE re.is_enable = 1 AND mday >= ? AND mday <= ? GROUP BY projectid, mday ) AS a GROUP BY projectid");
parameters.add(query.getStartMday());
parameters.add(query.getEndMday());
```