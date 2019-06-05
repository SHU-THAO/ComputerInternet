# 超市订单管理系统

## 分页查询

步骤：得到总个数-->分页-->计算当前页的起止与结束-->传入相关数据得到当前页的内容

1. select count(*) from user 

   <where> 

   <if > and username=#{userName} </if>
   <if > and userRole=#{userRole} </if>

   </where>

2. int start = (currentPageNo-1)*pageSize+1;

   userList = userMapper.getUserList(queryUserName, queryUserRole, start, pageSize);

3. select * from user

   <where> 

   <if > and username=#{userName} </if>
   <if > and userRole=#{userRole} </if>

   </where>

   limit #{start}, #{num}