{% data variables.product.prodname_registry %} is available with {% data variables.product.prodname_free_user %}, {% data variables.product.prodname_pro %}, {% data variables.product.prodname_free_team %} for organizations, {% data variables.product.prodname_team %}, {% data variables.product.prodname_ghe_cloud %}, {% data variables.product.prodname_ghe_server %} 3.0 or higher, and {% data variables.product.prodname_ghe_managed %}.{% ifversion ghes %} For more information about upgrading your {% data variables.product.prodname_ghe_server %} instance, see "[About upgrades to new releases](/admin/overview/about-upgrades-to-new-releases)" and refer to the [{% data variables.enterprise.upgrade_assistant %}](https://support.github.com/enterprise/server-upgrade) to find the upgrade path from your current release version.{% endif %}
{% ifversion fpt or ghec %}
<br>
{% data variables.product.prodname_registry %}は、レガシーのリポジトリごとのプランを使っているアカウントが所有しているプライベートリポジトリでは利用できません。 また、レガシーのリポジトリごとのプランを使っているアカウントは、リポジトリごとに課金されるため、{% data variables.product.prodname_container_registry %}にはアクセスできません。 {% data reusables.gated-features.more-info %}
{% endif %}