# 在 Canvas by Instructure 中设置 Claude LTI

本文提供了有关如何在Canvas LMS中启用Claude LTI集成的信息。这些步骤适用于Claude for Education管理员和学习管理系统(LMS)管理员。

# 在Canvas中创建Claude LTI开发者密钥

1. 在Canvas中，以管理员身份登录并导航至**Admin -> Developer Keys**。

2. 点击"+ Developer Key"，然后点击"+ LTI Key"。

3. 输入以下信息：

  1. **Key Name:** Claude LTI

  2. **Description:** 为Canvas LTI 1.3应用输入简短描述

  3. **Redirect URIs:** <https://claude.ai/lti/launch>

  4. **Title:** Claude LTI

  5. **Target Link URI:** <https://claude.ai/lti/launch>

  6. **OpenID Connect Initiation Url:** <https://claude.ai/api/lti/login>

  7. **JWK method:** <https://claude.ai/api/lti/keys>

4. 在**Additional Settings**下，将Privacy Level切换为**Public**。

5. 在**Placements**下，我们建议删除默认选项，并添加"Course Navigation"和"Assignment Edit"作为选项。

6. 点击"Save"。

7. 将状态切换为**On**。

# 将Claude LTI安装为应用

1. 在Canvas中，转到Admin -> Settings -> Apps。

2. 点击"View App Configurations"，然后选择"+ App"。

3. 选择**Configuration Type**"By Client ID"。

4. 输入为开发者密钥生成的Client ID（来自在Canvas中创建Claude LTI开发者密钥的第6步）。

5. 点击"Install"并刷新课程页面。

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/1611422430/c8e0875feac1f2c7cb033be74fc9/AD_4nXfLU_bui3EXcCjQ0qm70HD97neqjGayKeDer_t76utlci8gZSUjYRhw6ZSOlDdqSEcwXBzd_shAh7pQEJ-8OoE0O21DM5coOgxmO_WD5hlwiuwtS2iYXcTavhIRyQT5zKFWvfn3NA?expires=1794465900&amp;signature=0fd42da1fbee090d7af29b94aeda49299f24afe4addb0c04fb38f90b457ba76f&amp;req=dSYmF818n4VcWfMW1HO4zTEDa%2B8anfaIEv2ojHLMylawrvaSY4yCEqKHeyij%0AMhioygkpWbO4aH2OExs%3D%0A)

# 在Claude for Education组织设置中启用Claude LTI集成

1. 在Claude for Education中，以管理员身份登录。

2. 导航至**[Organization settings > Connectors](https://claude.ai/admin-settings/connectors)**。

3. 找到**Canvas**并点击"Enable"。

4. 在弹出的设置模态框中，输入所需信息以启用集成

  1. **Canvas Domain**

  2. **Client ID**（在Canvas Admin -> Developer Keys中找到）

  3. **Deployment ID**（在Canvas Admin -> Settings -> Apps -> View App Configurations -> Claude LTI Settings Button -> Deployment ID中找到）

5. 点击"Save Changes"。集成现在应该显示为已启用。

# 问题

如果您对Claude for Education计划账户或Claude LTI有任何疑问，我们建议您联系您所在大学的管理员。