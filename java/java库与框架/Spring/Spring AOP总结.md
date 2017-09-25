## Spring AOP

![](image/aop1.png)

一张图总结。

<table  cellpadding="2" cellspacing="0" border="1" bordercolor="#000000">
 <tbody>
  <tr>
   <td> 增强类型 </td>
   <td> 基于 AOP 接口 </td>
   <td> 基于 @Aspect </td>
   <td> 基于&nbsp;&lt;aop:config&gt; </td>
  </tr>
  <tr>
   <td> Before Advice（前置增强）<br> </td>
   <td> MethodBeforeAdvice<br> </td>
   <td> @Before<br> </td>
   <td> <span style="font-size:12px;line-height:18px;">&lt;</span><span style="font-size:12px;line-height:18px;">aop:before&gt;</span><br> </td>
  </tr>
  <tr>
   <td> AfterAdvice（后置增强）<br> </td>
   <td> AfterReturningAdvice<br> </td>
   <td> @After<br> </td>
   <td> <span style="font-size:12px;line-height:18px;">&lt;aop:after</span><span style="font-size:12px;line-height:18px;">&gt;</span><br> </td>
  </tr>
  <tr>
   <td> AroundAdvice（环绕增强）<br> </td>
   <td> MethodInterceptor<br> </td>
   <td> @Around<br> </td>
   <td> <span style="font-size:12px;line-height:18px;">&lt;aop:around&gt;</span><br> </td>
  </tr>
  <tr>
   <td> ThrowsAdvice（抛出增强<br> </td>
   <td> ThrowsAdvice<br> </td>
   <td> @AfterThrowing<br> </td>
   <td> &lt;aop:after-throwing&gt;<br> </td>
  </tr>
  <tr>
   <td> IntroductionAdvice（引入增强）<br> </td>
   <td> DelegatingIntroductionInterceptor<br> </td>
   <td> @DeclareParents<br> </td>
   <td> &lt;aop:declare-parents&gt;<br> </td>
  </tr>
 </tbody>
</table>


最后给一张 UML 类图描述一下 Spring AOP 的整体架构：

![](image/aop2.png)
