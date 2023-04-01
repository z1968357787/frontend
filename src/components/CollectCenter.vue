<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>采集中心</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-row>
        <el-col :span="9" style="line-height: 40px">
          <el-input v-model="queryDeployParams.queryInfo" @keyup.enter.native="queryDeploy" placeholder="Please input" class="input-with-select">
            <template slot="prepend">
              <el-select v-model="queryDeployParams.selectLabel" placeholder="Select" style="width: 110px">
                <el-option label="应用名" value="应用名" style="text-align: center"></el-option>
                <el-option label="部署节点" value="部署节点" style="text-align: center"></el-option>
<!--                <el-option label="部署集群" value="部署集群" style="text-align: center"></el-option>-->
                <el-option label="部署目录" value="部署目录" style="text-align: center"></el-option>
                <el-option label="部署时间" value="部署时间" style="text-align: center"></el-option>
              </el-select>
            </template>
            <el-button @click="queryDeploy" icon="el-icon-search" slot="append"></el-button>
          </el-input>
        </el-col>
        <el-col :span="3" style="line-height: 40px;" :offset="10">
          <el-button type="primary" size="medium" plain @click="addDeployButtonClick">应用部署</el-button>
        </el-col>
        <el-col :span="2" style="line-height: 40px">
          <el-button type="primary" size="medium" plain @click="deleteDeployBatch">批量删除</el-button>
        </el-col>
      </el-row>

      <el-table :data="deployList" style="width: 100%; margin-top: 20px" :header-row-style="{'height': '60px'}" @selection-change="handleSelectionChange" border stripe>
        <el-table-column type="selection" width="55" align="center" />
        <el-table-column label="应用名" width="190" align="center">
          <template #default="scope">{{ scope.row.deploy.application_name }}</template>
        </el-table-column>
        <el-table-column label="部署节点" width="250" align="center">
          <template #default="scope">{{ scope.row.node }}</template>
        </el-table-column>
        <el-table-column label="部署目录" width="300" align="center">
          <template #default="scope">{{ scope.row.deploy.directory }}</template>
        </el-table-column>
        <el-table-column label="部署时间" align="center">
          <template #default="scope">
            <i class="el-icon-time"></i>
            <!-- <span style="margin-left: 10px">{{ scope.row.deploy.created }}</span> -->
            <span style="margin-left: 10px">{{ deployCreatedList[scope.$index] }}</span>
          </template>
        </el-table-column>
<!--        <el-table-column label="操作" align="center">-->
<!--          <template #default="scope">-->
<!--            <el-button type="primary" icon="el-icon-delete" size="mini" plain @click="openDeleteDeployBox(scope.$index)"></el-button>-->
<!--          </template>-->
<!--        </el-table-column>-->
      </el-table>

      <el-pagination
        v-model:currentPage="getDeployParams.currentPage"
        :page-sizes="[1, 3, 5, 10]"
        :page-size="getDeployParams.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="deployAmount"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        style="margin-top: 20px"
      >
      </el-pagination>
    </el-card>

    <el-dialog title="应用部署" width="90%" height="50%" style="text-align: left" :visible.sync="addDeployDialogVisible">
      <div class="dialogDiv">
        <el-tree show-checkbox default-expand-all check-strictly>
        </el-tree>
      </div>
      <el-form v-for="addDeployForm in addDeployForms" :key="addDeployForm.id" :inline="true" ref="addDeployFormRef" :model="addDeployForm" :rules="addDeployFormRules" >
<!--        <div v-for="(item,index) in addDeployForm.dynamicItem" :key="index">-->
        <el-form-item label="部署节点" >
          <el-select v-model="addDeployForm.nodeId" placeholder="ip" :style="addDeployForm.type === '服务器' ? 'margin-left: 18px; width: 80%' : 'margin-left: 18px; width: 80%'">
            <el-option v-for="node in nodes" :key="node.id" :label="node.value" :value="node.id" style="text-align: center"></el-option>
          </el-select>
        </el-form-item>
        <el-button type="primary" size="medium" style="margin-left: 0px;margin-right: 10px" plain @click="addServerDialogVisible = true">添加服务器</el-button>
        <el-form-item v-if="addDeployForm.type === '服务器'" label="部署目录" prop="directory">
          <el-input v-model="addDeployForm.directory" style="width: 120%"></el-input>
        </el-form-item>
        <el-form-item >
          <el-upload ref="uploadRef"
                     :data="uploadData"
                     :action="uploadUrl"
                     :limit="1"
                     :auto-upload="false"
                     :on-change="uploadFileChange"
                     :on-success="uploadSuccess"
                     :before-remove="beforeRemove">
            <el-button slot="trigger" type="primary" size="small" style="margin-left: 100px" plain>选取应用</el-button>
            <div slot="tip" style="margin-left: 60px">只能上传小于600MB一个应用！</div>
          </el-upload>
        </el-form-item>
        <!--          <el-select v-model="addDeployForm.type" placeholder="Select" style="width: 110px" @change="deployFormTypeChange">-->
        <!--            <el-option label="服务器" value="服务器" style="text-align: center"></el-option>-->
        <!--            <el-option label="集群" value="集群" style="text-align: center"></el-option>-->
        <!--          </el-select>-->
        <el-form-item label="是否需要执行脚本" style="margin-left: 0px">
          <el-switch v-model="addDeployForm.is_execute" />
        </el-form-item>
        <el-form-item v-if="addDeployForm.is_execute" label="执行脚本">
          <el-upload ref="scriptServerRef"
                     :limit="1"
                     :data="scriptServerComputedData"
                     :action="scriptServerUrl"
                     :on-success="scriptServerSuccess"
                     :on-change="scriptServerChange">
            <el-button slot="trigger" type="primary" size="small" plain>选取</el-button>
          </el-upload>
          <codemirror v-if="addDeployForm.is_execute & scriptVisible" ref="codemirrorScriptServerRef"
                      style="width: 85%; margin-top: 20px; line-height: 150%; font-size: 14px"
                      :value="scriptServerCode"
                      :options="scriptOptions"
                      class="code"
                      @changes="updateMirrorCode">
          </codemirror>
        </el-form-item>
      </el-form>

<!--          <el-upload ref="uploadRef"-->
<!--                     :data="uploadData"-->
<!--                     :action="uploadUrl"-->
<!--                     :limit="1"-->
<!--                     :auto-upload="false"-->
<!--                     :on-change="uploadFileChange"-->
<!--                     :on-success="uploadSuccess"-->
<!--                     :before-remove="beforeRemove">-->
<!--            <el-button slot="trigger" type="primary" size="small" plain>选取应用</el-button>-->
<!--            <div slot="tip">只能上传一个应用，大小不能超过600MB！</div>-->
<!--          </el-upload>-->
          <!--          <el-button-->
          <!--            v-if="addDeployForm.type === '集群'"-->
          <!--            type="primary" size="medium" plain style="margin-left: 18px"-->
          <!--            @click="addClusterButtonClick">新建集群</el-button>-->
<!--        <el-form-item label="应用部署">-->
<!--          <el-upload ref="uploadRef"-->
<!--                     :data="uploadData"-->
<!--                     :action="uploadUrl"-->
<!--                     :limit="1"-->
<!--                     :auto-upload="false"-->
<!--                     :on-change="uploadFileChange"-->
<!--                     :on-success="uploadSuccess"-->
<!--                     :before-remove="beforeRemove">-->
<!--            <el-button slot="trigger" type="primary" size="small" plain>选取应用</el-button>-->
<!--            <div slot="tip">只能上传一个应用，大小不能超过600MB！</div>-->
<!--          </el-upload>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="部署节点(服务器）">-->
<!--&lt;!&ndash;          <el-select v-model="addDeployForm.type" placeholder="Select" style="width: 110px" @change="deployFormTypeChange">&ndash;&gt;-->
<!--&lt;!&ndash;            <el-option label="服务器" value="服务器" style="text-align: center"></el-option>&ndash;&gt;-->
<!--&lt;!&ndash;            <el-option label="集群" value="集群" style="text-align: center"></el-option>&ndash;&gt;-->
<!--&lt;!&ndash;          </el-select>&ndash;&gt;-->
<!--          <el-select v-model="addDeployForm.nodeId" placeholder="Select" :style="addDeployForm.type === '服务器' ? 'margin-left: 18px; width: 62.5%' : 'margin-left: 18px; width: 42.5%'">-->
<!--            <el-option v-for="node in nodes" :key="node.id" :label="node.value" :value="node.id" style="text-align: center"></el-option>-->
<!--          </el-select>-->
<!--&lt;!&ndash;          <el-button&ndash;&gt;-->
<!--&lt;!&ndash;            v-if="addDeployForm.type === '集群'"&ndash;&gt;-->
<!--&lt;!&ndash;            type="primary" size="medium" plain style="margin-left: 18px"&ndash;&gt;-->
<!--&lt;!&ndash;            @click="addClusterButtonClick">新建集群</el-button>&ndash;&gt;-->
<!--        </el-form-item>-->
<!--        <el-form-item v-if="addDeployForm.type === '服务器'" label="部署目录" prop="directory">-->
<!--          <el-input v-model="addDeployForm.directory" style="width: 85%"></el-input>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="是否需要执行脚本" style="margin-left: 0px">-->
<!--          <el-switch v-model="addDeployForm.is_execute" />-->
<!--        </el-form-item>-->
<!--        <el-form-item v-if="addDeployForm.is_execute" label="执行脚本">-->
<!--          <el-upload ref="scriptServerRef"-->
<!--                     :limit="1"-->
<!--                     :data="scriptServerComputedData"-->
<!--                     :action="scriptServerUrl"-->
<!--                     :on-success="scriptServerSuccess"-->
<!--                     :on-change="scriptServerChange">-->
<!--            <el-button slot="trigger" type="primary" size="small" plain>选取</el-button>-->
<!--          </el-upload>-->
<!--          <codemirror v-if="addDeployForm.is_execute & scriptVisible" ref="codemirrorScriptServerRef"-->
<!--                      style="width: 85%; margin-top: 20px; line-height: 150%; font-size: 14px"-->
<!--                      :value="scriptServerCode"-->
<!--                      :options="scriptOptions"-->
<!--                      @changes="updateMirrorCode">-->
<!--          </codemirror>-->
<!--        </el-form-item>-->
<!--      <el-form :inline="true" ref="addDeployFormRef" :model="addDeployForm" :rules="addDeployFormRules">-->
<!--        <el-form-item label="部署节点">-->
<!--          &lt;!&ndash;          <el-select v-model="addDeployForm.type" placeholder="Select" style="width: 110px" @change="deployFormTypeChange">&ndash;&gt;-->
<!--          &lt;!&ndash;            <el-option label="服务器" value="服务器" style="text-align: center"></el-option>&ndash;&gt;-->
<!--          &lt;!&ndash;            <el-option label="集群" value="集群" style="text-align: center"></el-option>&ndash;&gt;-->
<!--          &lt;!&ndash;          </el-select>&ndash;&gt;-->
<!--          <el-select v-model="addDeployForm.nodeId" placeholder="Select" :style="addDeployForm.type === '服务器' ? 'margin-left: 0px; width: 25%' : 'margin-left: 0px; width: 10%'">-->
<!--            <el-option v-for="node in nodes" :key="node.id" :label="node.value" :value="node.id" style="text-align: center"></el-option>-->
<!--          </el-select>-->
<!--          <el-form-item v-if="addDeployForm.type === '服务器'" label="部署目录" prop="directory" style="margin-left: 5px">-->
<!--            <el-input v-model="addDeployForm.directory" style="width:120%"></el-input>-->
<!--          </el-form-item >-->
<!--          <el-form-item >-->
<!--            <el-upload ref="uploadRef"-->
<!--                       :data="uploadData"-->
<!--                       :action="uploadUrl"-->
<!--                       :limit="1"-->
<!--                       :auto-upload="false"-->
<!--                       :on-change="uploadFileChange"-->
<!--                       :on-success="uploadSuccess"-->
<!--                       :before-remove="beforeRemove">-->
<!--              <el-button slot="trigger" type="primary" size="small" style="margin-left: 50px" plain>选取应用</el-button>-->
<!--              <div slot="tip" style="margin-left: 10px">只能上传小于600MB一个应用！</div>-->
<!--            </el-upload>-->
<!--          </el-form-item>-->
<!--          &lt;!&ndash;          <el-upload ref="uploadRef"&ndash;&gt;-->
<!--          &lt;!&ndash;                     :data="uploadData"&ndash;&gt;-->
<!--          &lt;!&ndash;                     :action="uploadUrl"&ndash;&gt;-->
<!--          &lt;!&ndash;                     :limit="1"&ndash;&gt;-->
<!--          &lt;!&ndash;                     :auto-upload="false"&ndash;&gt;-->
<!--          &lt;!&ndash;                     :on-change="uploadFileChange"&ndash;&gt;-->
<!--          &lt;!&ndash;                     :on-success="uploadSuccess"&ndash;&gt;-->
<!--          &lt;!&ndash;                     :before-remove="beforeRemove">&ndash;&gt;-->
<!--          &lt;!&ndash;            <el-button slot="trigger" type="primary" size="small" plain>选取应用</el-button>&ndash;&gt;-->
<!--          &lt;!&ndash;            <div slot="tip">只能上传一个应用，大小不能超过600MB！</div>&ndash;&gt;-->
<!--          &lt;!&ndash;          </el-upload>&ndash;&gt;-->
<!--          &lt;!&ndash;          <el-button&ndash;&gt;-->
<!--          &lt;!&ndash;            v-if="addDeployForm.type === '集群'"&ndash;&gt;-->
<!--          &lt;!&ndash;            type="primary" size="medium" plain style="margin-left: 18px"&ndash;&gt;-->
<!--          &lt;!&ndash;            @click="addClusterButtonClick">新建集群</el-button>&ndash;&gt;-->
<!--        </el-form-item>-->
<!--        &lt;!&ndash;        <el-form-item label="应用部署">&ndash;&gt;-->
<!--        &lt;!&ndash;          <el-upload ref="uploadRef"&ndash;&gt;-->
<!--        &lt;!&ndash;                     :data="uploadData"&ndash;&gt;-->
<!--        &lt;!&ndash;                     :action="uploadUrl"&ndash;&gt;-->
<!--        &lt;!&ndash;                     :limit="1"&ndash;&gt;-->
<!--        &lt;!&ndash;                     :auto-upload="false"&ndash;&gt;-->
<!--        &lt;!&ndash;                     :on-change="uploadFileChange"&ndash;&gt;-->
<!--        &lt;!&ndash;                     :on-success="uploadSuccess"&ndash;&gt;-->
<!--        &lt;!&ndash;                     :before-remove="beforeRemove">&ndash;&gt;-->
<!--        &lt;!&ndash;            <el-button slot="trigger" type="primary" size="small" plain>选取应用</el-button>&ndash;&gt;-->
<!--        &lt;!&ndash;            <div slot="tip">只能上传一个应用，大小不能超过600MB！</div>&ndash;&gt;-->
<!--        &lt;!&ndash;          </el-upload>&ndash;&gt;-->
<!--        &lt;!&ndash;        </el-form-item>&ndash;&gt;-->
<!--                <el-form-item label="部署节点(服务器）">-->
<!--        &lt;!&ndash;          <el-select v-model="addDeployForm.type" placeholder="Select" style="width: 110px" @change="deployFormTypeChange">&ndash;&gt;-->
<!--        &lt;!&ndash;            <el-option label="服务器" value="服务器" style="text-align: center"></el-option>&ndash;&gt;-->
<!--        &lt;!&ndash;            <el-option label="集群" value="集群" style="text-align: center"></el-option>&ndash;&gt;-->
<!--        &lt;!&ndash;          </el-select>&ndash;&gt;-->
<!--                  <el-select v-model="addDeployForm.nodeId" placeholder="Select" :style="addDeployForm.type === '服务器' ? 'margin-left: 18px; width: 62.5%' : 'margin-left: 18px; width: 42.5%'">-->
<!--                    <el-option v-for="node in nodes" :key="node.id" :label="node.value" :value="node.id" style="text-align: center"></el-option>-->
<!--                  </el-select>-->
<!--        &lt;!&ndash;          <el-button&ndash;&gt;-->
<!--        &lt;!&ndash;            v-if="addDeployForm.type === '集群'"&ndash;&gt;-->
<!--        &lt;!&ndash;            type="primary" size="medium" plain style="margin-left: 18px"&ndash;&gt;-->
<!--        &lt;!&ndash;            @click="addClusterButtonClick">新建集群</el-button>&ndash;&gt;-->
<!--                </el-form-item>-->
<!--        &lt;!&ndash;        <el-form-item v-if="addDeployForm.type === '服务器'" label="部署目录" prop="directory">&ndash;&gt;-->
<!--        &lt;!&ndash;          <el-input v-model="addDeployForm.directory" style="width: 85%"></el-input>&ndash;&gt;-->
<!--        &lt;!&ndash;        </el-form-item>&ndash;&gt;-->
<!--        <el-form-item label="是否需要执行脚本" style="margin-left: 0px">-->
<!--          <el-switch v-model="addDeployForm.is_execute" />-->
<!--        </el-form-item>-->
<!--        <el-form-item v-if="addDeployForm.is_execute" label="执行脚本">-->
<!--          <el-upload ref="scriptServerRef"-->
<!--                     :limit="1"-->
<!--                     :data="scriptServerComputedData"-->
<!--                     :action="scriptServerUrl"-->
<!--                     :on-success="scriptServerSuccess"-->
<!--                     :on-change="scriptServerChange">-->
<!--            <el-button slot="trigger" type="primary" size="small" plain>选取</el-button>-->
<!--          </el-upload>-->
<!--          <codemirror v-if="addDeployForm.is_execute & scriptVisible" ref="codemirrorScriptServerRef"-->
<!--                      style="width: 85%; margin-top: 20px; line-height: 150%; font-size: 14px"-->
<!--                      :value="scriptServerCode"-->
<!--                      :options="scriptOptions"-->
<!--                      @changes="updateMirrorCode">-->
<!--          </codemirror>-->
<!--        </el-form-item>-->
<!--      </el-form>-->
      <template slot="footer">
        <span class="dialog-footer">
          <el-button icon="el-icon-circle-plus-outline" @click="addDeployment">新增部署项</el-button>
          <el-button icon="el-icon-remove-outline" @click="deleteDeployment" :disabled="flag">删除部署项</el-button>
          <el-button type="primary" @click="addDeploy">部署</el-button>
          <el-button @click="cancelAddDeployForm">取消</el-button>
          <el-button @click="resetAddDeployForm">重置</el-button>
        </span>
      </template>
    </el-dialog>
    <el-dialog title="添加服务器" style="text-align: left" :visible.sync="addServerDialogVisible">
      <el-form ref="addServerFormRef" :model="addServerForm" :rules="addServerFormRules" label-width="150px">
        <el-form-item label="IP 地址" prop="ip">
          <el-input v-model="addServerForm.ip" style="width: 85%"></el-input>
        </el-form-item>
        <el-form-item label="端口" prop="port">
          <el-input v-model="addServerForm.port" style="width: 85%"></el-input>
        </el-form-item>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addServerForm.username" style="width: 85%"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addServerForm.password" style="width: 85%" show-password></el-input>
        </el-form-item>
        <el-form-item label="测试连接">
          <el-button type="primary" icon="el-icon-s-promotion" size="mini" plain @click="testWhenAddServer"></el-button>
        </el-form-item>
        <el-form-item label="使用默认采集工具">
          <el-switch v-model="addServerForm.is_default"></el-switch>
        </el-form-item>
        <el-form-item v-if="!addServerForm.is_default" label="采集工具上传">
          <el-upload ref="uploadRef"
                     :data="uploadData"
                     :action="uploadUrl"
                     :limit="1"
                     :auto-upload="false"
                     :on-success="uploadSuccess"
                     :before-remove="beforeRemove">
            <el-button slot="trigger" type="primary" size="small" plain>选取</el-button>
            <div slot="tip">只能上传一个采集工具，大小不能超过100MB！</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <!-- <el-progress v-if="addServerProgress" :percentage="0" :format="addServerFormat" :indeterminate="true" /> -->
      <template slot="footer">
        <span class="dialog-footer">
          <el-button type="primary" @click="addServer">创建</el-button>
          <el-button @click="cancelAddServerForm">取消</el-button>
          <el-button @click="resetAddServerForm">重置</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 新建集群 -->
<!--    <el-dialog title="新建集群" style="text-align: left" :visible.sync="addClusterDialogVisible">-->
<!--      <el-form ref="addClusterFormRef" :model="addClusterForm" :rules="addClusterFormRules" label-width="150px">-->
<!--        <el-form-item label="名称" prop="name">-->
<!--          <el-input v-model="addClusterForm.name" style="width: 85%"></el-input>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="描述" prop="description">-->
<!--          <el-input v-model="addClusterForm.description" style="width: 85%"></el-input>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="服务器列表">-->
<!--          <el-table ref="addClusterTableRef" :data="serverList" style="width: 85%; margin-top: 15px" :header-row-style="{'height': '60px'}" @selection-change="selectServerChange" border stripe>-->
<!--            <el-table-column type="selection" width="55" align="center" />-->
<!--            <el-table-column label="IP" width="240" align="center">-->
<!--              <template #default="scope">{{ scope.row.ip }}</template>-->
<!--            </el-table-column>-->
<!--            <el-table-column label="用户名" align="center">-->
<!--              <template #default="scope">{{ scope.row.username }}</template>-->
<!--            </el-table-column>-->
<!--          </el-table>-->
<!--        </el-form-item>-->
<!--      </el-form>-->
<!--      <template slot="footer">-->
<!--        <span class="dialog-footer">-->
<!--          <el-button type="primary" @click="addCluster">创建</el-button>-->
<!--          <el-button @click="cancelAddClusterForm">取消</el-button>-->
<!--          <el-button @click="resetAddClusterForm">重置</el-button>-->
<!--        </span>-->
<!--      </template>-->
<!--    </el-dialog>-->

    <!-- 采集模块 -->
    <el-card class="box-card" style="margin-top: 20px">
      <el-row>
        <el-col :span="9" style="line-height: 40px">
          <el-input v-model="queryCollectParams.queryInfo" @keyup.enter.native="queryCollect" placeholder="Please input" class="input-with-select">
            <template slot="prepend">
              <el-select v-model="queryCollectParams.selectLabel" placeholder="Select" style="width: 110px">
                <el-option label="采集任务" value="采集任务" style="text-align: center"></el-option>
<!--                <el-option label="应用名" value="应用名" style="text-align: center"></el-option>-->
                <el-option label="部署节点" value="部署节点" style="text-align: center"></el-option>
<!--                <el-option label="部署集群" value="部署集群" style="text-align: center"></el-option>-->
                <el-option label="配置信息" value="配置信息" style="text-align: center"></el-option>
                <el-option label="用例类名" value="用例类名" style="text-align: center"></el-option>
                <el-option label="采集时间" value="采集时间" style="text-align: center"></el-option>
              </el-select>
            </template>
            <el-button @click="queryCollect" icon="el-icon-search" slot="append"></el-button>
          </el-input>
        </el-col>
        <el-col :span="3" style="line-height: 40px;" :offset="8">
          <el-button type="primary" size="medium" plain @click="addCollectDialogVisible = true;">新建采集任务</el-button>
        </el-col>
        <el-col :span="2" style="line-height: 40px">
          <el-button type="primary" size="medium" plain @click="CollectBatch">批量采集</el-button>
        </el-col>
        <el-col :span="2" style="line-height: 40px">
          <el-button type="primary" size="medium" plain @click="deleteCollectBatch">批量删除</el-button>
        </el-col>
      </el-row>

      <el-table :data="collectList" style="width: 100%; margin-top: 20px" :header-row-style="{'height': '60px'}" @selection-change="handleCollectSelectionChange" border stripe>
        <el-table-column type="selection" width="55" align="center" />
        <el-table-column label="采集任务" width="120" align="center">
          <template #default="scope">{{ scope.row.collect.name }}</template>
        </el-table-column>
<!--        <el-table-column label="应用名" width="120" align="center">-->
<!--          <template #default="scope">{{ scope.row.application_name }}</template>-->
<!--        </el-table-column>-->
        <el-table-column label="日志采集类型" width="120" align="center">
          <template #default="scope">{{ scope.row.collect.type }}</template>
        </el-table-column>
        <el-table-column label="部署节点" width="150" align="center">
          <template #default="scope">{{ scope.row.node }}</template>
        </el-table-column>
        <el-table-column label="配置信息" width="140" align="center">
          <template #default="scope">
            <el-link @click="previewAgentConfig(scope.row.collect.id)" type="primary">agent-config.xml</el-link>
            <el-link @click="previewConfig(scope.row.collect.id)" type="primary">config.xml</el-link>
          </template>
        </el-table-column>
        <el-table-column label="用例类名" width="120" align="center">
          <template #default="scope">{{ scope.row.collect.use_case_name }}</template>
        </el-table-column>
        <el-table-column label="测试用例脚本" width="140" align="center">
          <template #default="scope">
            <el-link @click="previewAgentConfig(scope.row.collect.id)" type="primary">agent-config.xml</el-link>
            <el-link @click="previewConfig(scope.row.collect.id)" type="primary">config.xml</el-link>
          </template>
        </el-table-column>
        <el-table-column label="采集时间" width="130" align="center">
          <template #default="scope">
            <i class="el-icon-time"></i>
            <span style="margin-left: 10px">{{ collectCreatedList[scope.$index] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="日志下载" width="120" align="center">
          <template #default="scope">
            <div v-if="scope.row.collect.is_build">
              <el-link @click="downloadLog(scope.row.collect.id)" type="primary">{{ scope.row.collect.log_name }}</el-link>
            </div>
            <div v-else>待采集</div>
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center">
          <template #default="scope">
            <el-button type="primary" icon="el-icon-video-play" size="mini" plain @click="buildCollect(scope.row.collect.id)"></el-button>
            <el-button type="primary" icon="el-icon-switch-button" size="mini" plain @click="overCollect(scope.row.collect.id)"></el-button>
            <el-button type="primary" icon="el-icon-delete" size="mini" plain @click="openDeleteCollectBox(scope.$index)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        v-model:currentPage="getCollectParams.currentPage"
        :page-sizes="[1, 3, 5, 10]"
        :page-size="getCollectParams.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="collectAmount"
        @size-change="collectSizeChange"
        @current-change="collectCurrentChange"
        style="margin-top: 20px"
      >
      </el-pagination>
    </el-card>

    <el-dialog title="新建采集任务" style="text-align: left" :visible.sync="addCollectDialogVisible">
      <el-form ref="addCollectFormRef" :model="addCollectForm" :rules="addCollectFormRules" label-width="150px">
        <el-row>
          <el-col :span="13">
            <el-form-item label="采集任务名称" prop="name">
              <el-input v-model="addCollectForm.name" style="width: 85%"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="采集类型选择">
              <el-select v-model="addCollectForm.pattern" placeholder="请选择">
                <el-option v-for="item in options"
                           :key="item.pattern"
                           :label="item.label"
                           :value="item.pattern"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <!--
        <el-form-item label="部署应用id" prop="deploy_id">
          <el-select v-model="addCollectForm.deploy_id" placeholder="Select" @change="getCollectFormDetail(addCollectForm.deploy_id)">
            <el-option v-for="deployMap in deployList" :key="deployMap.deploy.id" :label="deployMap.deploy.id" :value="deployMap.deploy.id" style="text-align: center"></el-option>
          </el-select>
        </el-form-item>
        -->
        <el-form-item label="应用名_部署节点" prop="deploy_id">
          <el-select v-model="addCollectForm.deploy_id" placeholder="Select" @change="getCollectFormDetail(addCollectForm.deploy_id)" style="display: block; width: 85%">
            <el-option v-for="deployMap in deployList" :key="deployMap.deploy.id" :label="deployMap.deploy.application_name + '_' + deployMap.node" :value="deployMap.deploy.id" style="text-align: center"></el-option>
          </el-select>
        </el-form-item>
        <!--
        <el-form-item label="应用名">
          <el-input disabled v-model="selectApplicationDetail.application_name" style="width: 85%"></el-input>
        </el-form-item>
        <el-form-item label="部署服务器/集群">
          <el-input disabled v-model="selectApplicationDetail.node" style="width: 85%"></el-input>
        </el-form-item>
        -->
        <el-form-item label="部署目录">
          <el-input disabled v-model="selectApplicationDetail.directory" style="width: 85%"></el-input>
        </el-form-item>
<!--        <el-form-item label="部署时间">-->
<!--          <el-input disabled v-model="selectApplicationDetail.created" style="width: 85%"></el-input>-->
<!--        </el-form-item>-->
        <el-row>
          <el-col :span="2">
            <el-form-item label="是否使用用例脚本" style="margin-left: 0px">
              <el-switch v-model="addCollectForm.is_execute" />
            </el-form-item>
          </el-col>
          <el-col :span="4">
            <el-form-item v-if="addCollectForm.is_execute">
              <el-upload ref="scriptServerRef"
                         :limit="1"
                         :data="scriptServerComputedData"
                         :action="scriptServerUrl"
                         :on-success="scriptServerSuccess"
                         :on-change="scriptServerChange">
                <el-button slot="trigger" type="primary" size="small" plain>选取脚本</el-button>
              </el-upload>
              <codemirror v-if="this.addDeployForms[this.addDeployForms.length - 1].is_execute && scriptVisible" ref="codemirrorScriptServerRef"
                          style="width: 85%; margin-top: 20px; line-height: 150%; font-size: 14px"
                          :value="scriptServerCode"
                          :options="scriptOptions"
                          @changes="updateMirrorCode">
              </codemirror>
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="是否使用默认配置">
          <el-switch v-model="addCollectForm.is_default" />
        </el-form-item>
        <div v-if="!addCollectForm.is_default">
          <el-form-item :label="(index === 0) ? '选择采集包名' : ''" v-for="(collectPackage, index) in addCollectForm.collectPackageList" :key="collectPackage.id">
            <el-input disabled :placeholder="collectPackage.name" style="width: 78%" />
            <el-checkbox v-model="collectPackage.is_collect" style="zoom: 150%; margin-left: 12px"></el-checkbox>
          </el-form-item>
        </div>
        <el-form-item v-if="!addCollectForm.is_default" label="日志文件名称">
          <el-input v-model="addCollectForm.log_name" style="width: 85%"></el-input>
        </el-form-item>
        <el-form-item label="用例类名">
          <el-input v-model="addCollectForm.use_case_name" style="width: 85%"></el-input>
        </el-form-item>
      </el-form>
      <template slot="footer">
        <span class="dialog-footer">
          <el-button type="primary" @click="addCollect">创建</el-button>
          <el-button @click="cancelAddCollectForm">取消</el-button>
          <el-button @click="resetAddCollectForm">重置</el-button>
        </span>
      </template>
    </el-dialog>

    <el-dialog title="agent-config.xml" style="text-align: left" :visible.sync="agentConfigDialogVisible">
      <codemirror v-if="agentConfigCodeVisible" ref="agentConfigCodeRef"
                  style="width: 100%; line-height: 150%; font-size: 14px"
                  :value="agentConfigCode"
                  :options="configOptions">
      </codemirror>
    </el-dialog>

    <el-dialog title="config.xml" style="text-align: left" :visible.sync="configDialogVisible">
      <codemirror v-if="configCodeVisible" ref="configCodeRef"
                  style="width: 100%; line-height: 150%; font-size: 14px"
                  :value="configCode"
                  :options="configOptions">
      </codemirror>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import Vue from 'vue'
import { VueCodemirror } from 'vue-codemirror'
import 'codemirror/lib/codemirror.css'
import 'codemirror/theme/ayu-mirage.css'
import 'codemirror/mode/shell/shell'
import 'codemirror/mode/xml/xml'

export default {
  name: 'CollectCenter',
  created () {
    setTimeout(() => {
      this.addDeployForms[this.addDeployForms.length - 1].project_id = this.$store.state.currentProjectId
      this.getNodes()
      this.getDeploy()
      this.getCollect()
    }, 100)
  },
  components: {
    // eslint-disable-next-line vue/no-unused-components
    VueCodemirror
  },
  data () {
    return {
      // codemirror (edit server script)
      scriptServerCode: '',
      scriptOptions: {
        tabSize: 4,
        mode: 'application/x-sh',
        // mode: 'application/xml',
        theme: 'ayu-mirage',
        lineNumbers: true,
        line: true,
        cursorHeight: 0.90
      },
      scriptVisible: false,
      // 连接服务器模块
      addServerDialogVisible: false,
      addServerForm: {
        ip: '',
        port: '',
        username: '',
        password: '',
        status: 'DISCONNECTED',
        is_default: false,
        project_id: 0
      },
      addServerFormRules: {
        ip: [
          {
            required: true,
            message: '请输入服务器IP地址',
            trigger: 'blur'
          }
        ],
        port: [
          {
            required: true,
            message: '请输入服务器端口号',
            trigger: 'blur'
          }
        ],
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          }
        ]
      },
      getServerParams: {
        currentPage: 1,
        pageSize: 5,
        project_id: 0
      },
      serverList: [],
      serverCreatedList: [],
      serverAmount: 0,
      tableSelection: [],
      queryServerParams: {
        selectLabel: 'IP',
        queryInfo: '',
        currentPage: 1,
        pageSize: 5,
        project_id: 0
      },
      addServerProgress: false,
      addServerFormat: '服务器添加中',
      // 部署模块
      nodes: [
        {}
      ],
      addDeployDialogVisible: false,
      addDeployForms: [{
        type: '服务器',
        nodeId: '',
        directory: '',
        project_id: this.$store.state.currentProjectId,
        is_executed: false,
        script: '',
        code: '',
        DeployVersion: '1.0.0'
      }],
      flag: true,
      addDeployFormRules: {
        directory: [
          {
            required: true,
            message: '请输入应用部署目录',
            trigger: 'blur'
          }
        ]
      },
      uploadUrl: axios.defaults.baseURL + 'collect/addDeploy',
      uploadFile: [],
      getDeployParams: {
        currentPage: 1,
        pageSize: 5,
        project_id: 0
      },
      deployList: [],
      deployCreatedList: [],
      deployAmount: 0,
      deploySelected: [],
      queryDeployParams: {
        selectLabel: '应用名',
        queryInfo: '',
        currentPage: 1,
        pageSize: 5,
        project_id: 0
      },
      scriptServerUrl: axios.defaults.baseURL + '/collect/uploadServerScript',
      scriptServerData: {
        project_id: 0,
        applicationName: ''
      },
      // 集群模块
      // addClusterDialogVisible: false,
      // addClusterForm: {
      //   name: '',
      //   description: '',
      //   project_id: 0,
      //   serverList: []
      // },
      // addClusterFormRules: {
      //   name: [
      //     { required: true, message: '请输入项目名称', trigger: 'blur' },
      //     { min: 1, max: 30, message: '长度在 1 到 30 个字符', trigger: 'blur' }
      //   ],
      //   description: [
      //     { required: true, message: '请输入项目描述', trigger: 'blur' },
      //     { min: 1, max: 50, message: '长度在 1 到 50 个字符', trigger: 'blur' }
      //   ]
      // },
      // serverList: [],
      // 采集模块
      addCollectDialogVisible: false,
      addCollectForm: {
        name: '',
        pattern: '',
        deploy_id: '请选择',
        is_default: true,
        log_name: '',
        use_case_name: '',
        project_id: 0,
        is_execute: false,
        collectPackageList: []
      },
      addCollectFormRules: {
        name: [
          {
            required: true,
            message: '请输入采集任务名称',
            trigger: 'blur'
          }
        ],
        deploy_id: [
          {
            required: true,
            message: '请选择部署应用id',
            trigger: 'blur'
          }
        ]
      },
      options: [{
        pattern: '选项1',
        label: '带参数'
      }, {
        pattern: '选项2',
        label: '不带参数'
      }],
      selectApplicationDetail: {},
      getCollectParams: {
        currentPage: 1,
        pageSize: 5,
        project_id: 0
      },
      collectList: [],
      collectCreatedList: [],
      collectAmount: 0,
      collectSelected: [],
      queryCollectParams: {
        selectLabel: '采集任务',
        queryInfo: '',
        currentPage: 1,
        pageSize: 5,
        project_id: 0
      },
      // CodeMirror agentConfig.xml & config.xml
      configOptions: {
        tabSize: 4,
        mode: 'application/xml',
        theme: 'ayu-mirage',
        lineNumbers: true,
        line: true,
        cursorHeight: 0.90,
        readOnly: true
      },
      agentConfigDialogVisible: false,
      agentConfigCodeVisible: false,
      agentConfigCode: '',
      configDialogVisible: false,
      configCodeVisible: false,
      configCode: ''
    }
  },
  methods: {
    // 连接服务器
    async addServer () {
      // this.addServerProgress = true
      if (this.addServerForm.is_default === true) {
        this.addServerForm.project_id = this.$store.state.currentProjectId
        await this.$http.post(
          'server/addServerWithoutFile',
          this.addServerForm,
          { headers: { Authorization: this.$store.getters.getToken } }
        )
        this.cancelAddServerForm()
        await this.getServer()
      } else {
        this.addServerForm.project_id = this.$store.state.currentProjectId
        await this.$refs.addServerFormRef.validate(async (valid) => {
          if (!valid) return false
          this.$nextTick(async () => {
            await this.$refs.uploadRef.submit()
            this.cancelAddServerForm()
          })
        })
      }
    },
    cancelAddServerForm () {
      this.resetAddServerForm()
      this.addServerDialogVisible = false
      // this.addServerProgress = false
    },
    resetAddServerForm () {
      this.addServerForm = {
        ip: '',
        port: '',
        username: '',
        password: '',
        status: 'DISCONNECTED',
        is_default: false,
        project_id: ''
      }
      this.$refs.addServerFormRef.resetFields()
      // this.$refs.uploadRef.clearFiles()
    },
    async getServer () {
      this.getServerParams.project_id = this.$store.state.currentProjectId
      const { data: res } = await this.$http.get(
        'server/getServer',
        { params: this.getServerParams, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.serverList = res.data
      if (this.serverList !== undefined && this.serverList !== null && this.serverList.length > 0) {
        this.serverAmount = this.serverList[0].serverAmount
      } else {
        this.serverAmount = 0
      }
      for (const item of this.serverList) {
        let created = item.server.created
        created = created.substring(0, 10) + ' ' + created.substring(11)
        this.serverCreatedList.push(created)
      }
    },
    // 部署模块
    addDeployButtonClick () {
      this.addDeployDialogVisible = true
      this.scriptServerData.project_id = this.$store.state.currentProjectId
    },
    async getNodes () {
      const { data: res } = await this.$http.get(
        'collect/getNodes',
        { params: { project_id: this.$store.state.currentProjectId, type: '服务器' }, headers: { Authorization: this.$store.getters.getToken } }
        // { params: { project_id: 2, type: this.addDeployForm.type }, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.nodes = res.data
    },
    deployFormTypeChange () {
      this.getNodes()
      this.addDeployForms[this.addDeployForms.length - 1].nodeId = '请选择'
      this.addDeployForms[this.addDeployForms.length - 1].is_execute = false
      this.scriptVisible = false
    },
    cancelAddDeployForm () {
      // this.resetAddDeployForm()
      this.addDeployDialogVisible = false
    },
    addDeployment () {
      const arr = {
        type: '服务器',
        nodeId: '',
        directory: '',
        project_id: this.$store.state.currentProjectId,
        is_executed: false,
        script: '',
        code: '',
        DeployVersion: ''
      }
      this.addDeployForms.push(arr)
      this.flags()
    },
    deleteDeployment () {
      this.addDeployForms.length = this.addDeployForms.length - 1
      this.flags()
    },
    flags () {
      if (this.addDeployForms.length < 2) {
        this.flag = true
      } else {
        this.flag = true
        this.flag = false
      }
    },
    resetAddDeployForm () {
      console.log('xxx')
      this.addDeployForms = [{
        type: '服务器',
        nodeId: '',
        directory: '',
        project_id: 0,
        is_executed: false,
        script: '',
        code: '',
        DeployVersion: ''
      }]
      this.flags()
      this.scriptServerCode = ''
      this.scriptVisible = false
      this.$refs.addDeployFormRef.resetFields()
      // this.$refs.uploadRef.clearFiles()
      // this.$refs.scriptServerRef.clearFiles()
    },
    async testWhenAddServer () {
      this.addServerForm.project_id = this.$store.state.currentProjectId
      const { data: res } = await this.$http.post(
        'server/testServer',
        this.addServerForm,
        { headers: { Authorization: this.$store.getters.getToken } }
      )
      if (res.data === 'CONNECTED') {
        Vue.prototype.$message({
          type: 'success',
          message: '连接测试成功'
        })
      } else {
        Vue.prototype.$message({
          type: 'error',
          message: '连接测试失败'
        })
      }
      this.addServerForm.status = res.data
    },
    // async addDeploy () {
    //   this.addDeployForms[this.addDeployForms.length - 1].code = this.scriptServerCode
    //   console.log(this.$refs.addDeployFormRef[0])
    //   await this.$refs.addDeployFormRef[0].validate(async (valid) => {
    //     if (!valid) return false
    //     this.$nextTick(async () => {
    //       //  console.log(JSON.stringify(this.$refs.uploadRef[0]))
    //       await this.$refs.uploadRef[0].submit()
    //       this.cancelAddDeployForm()
    //     })
    //   })
    //  },
    addDeploy () {
      this.addDeployForms[this.addDeployForms.length - 1].code = this.scriptServerCode
      console.log(this.$refs.addDeployFormRef[0])
      this.$refs.addDeployFormRef[0].validate(async (valid) => {
        if (!valid) return false
        this.$nextTick(async () => {
          console.log(this.$refs.uploadRef[0])
          await this.$refs.uploadRef[0].submit()
          this.cancelAddDeployForm()
        })
      })
    },
    uploadFileChange (file, fileList) {
      if (fileList && fileList.length > 0) {
        this.uploadFile = fileList
        this.scriptServerData.applicationName = this.uploadFile[0].name
      } else if (file && file.name) { this.scriptServerData.applicationName = file.name }
    },
    uploadSuccess () {
      this.getDeploy()
    },
    beforeRemove (file, fileList) {
      return Vue.prototype.$confirm(`确定取消上传 ${file.name} 吗?`)
    },
    async getDeploy () {
      this.getDeployParams.project_id = this.$store.state.currentProjectId
      const { data: res } = await this.$http.get(
        'collect/getDeploy',
        { params: this.getDeployParams, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.deployList = res.data
      if (this.deployList !== undefined && this.deployList !== null && this.deployList.length > 0) {
        this.deployAmount = this.deployList[0].deployAmount
      } else {
        this.deployAmount = 0
      }
      for (const item of this.deployList) {
        let created = item.deploy.created
        created = created.substring(0, 10) + ' ' + created.substring(11)
        this.deployCreatedList.push(created)
      }
    },
    handleSizeChange (newPageSize) {
      this.getDeployParams.currentPage = 1
      this.getDeployParams.pageSize = newPageSize
      this.getDeploy()
    },
    handleCurrentChange (newCurrentPage) {
      this.getDeployParams.currentPage = newCurrentPage
      this.getDeploy()
    },
    async deleteDeploy (index) {
      await this.$http.post(
        'collect/deleteDeploy',
        this.deployList[index].deploy,
        { headers: { Authorization: this.$store.getters.getToken } }
      )
      await this.getDeploy()
    },
    openDeleteDeployBox (index) {
      Vue.prototype.$confirm('此操作将撤销该应用部署, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.deleteDeploy(index)
      }).catch(() => {
        Vue.prototype.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    handleSelectionChange (selection) {
      console.log('selection: ' + JSON.stringify(selection))
      this.deploySelected = selection
    },
    async deleteDeployBatch () {
      await this.$http.post(
        'collect/deleteDeployBatch',
        this.deploySelected,
        { headers: { Authorization: this.$store.getters.getToken } }
      )
      await this.getDeploy()
    },
    async queryDeploy () {
      this.queryDeployParams.currentPage = this.getDeployParams.currentPage
      this.queryDeployParams.pageSize = this.getDeployParams.pageSize
      this.queryDeployParams.project_id = this.$store.state.currentProjectId
      const { data: res } = await this.$http.get(
        'collect/queryDeploy',
        { params: this.queryDeployParams, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.deployList = res.data
      if (this.deployList !== undefined && this.deployList !== null && this.deployList.length > 0) {
        this.deployAmount = this.deployList[0].deployAmount
      } else {
        this.deployAmount = 0
      }
    },
    scriptServerChange (file, fileList) {
      if (fileList.length > 0) {
        this.addDeployForms[this.addDeployForms.length - 1].script = fileList[0].name
      }
    },
    async scriptServerSuccess () {
      const id = this.$store.state.currentProjectId
      const { data: res } = await this.$http.get(
        'collect/getServerScript',
        { params: { project_id: id, applicationName: this.scriptServerData.applicationName, script: this.addDeployForms[this.addDeployForms.length - 1].script }, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.scriptServerCode = res.data
      this.scriptVisible = true
    },
    updateMirrorCode () {
      const codemirrorInstance = this.$refs.codemirrorScriptServerRef
      const newCode = codemirrorInstance.getValue()
      //  this.scriptServerCode = 'java -javaagent:"/home/yj/autodeploy/tool/log_agent-0.0.4-SNAPSHOT-shaded.jar"="/home/yj/autodeploy/config/josephus/agent-config.xml" -jar /home/yj/autodeploy/application/Josephus.jar'
      //  this.scriptServerCode = this.$refs.codemirrorScriptServerRef.codemirror.getValue()
      this.scriptServerCode = newCode
    },
    // 集群模块
    // selectServerChange (selection) {
    //   this.addClusterForm.serverList = selection
    // },
    // addClusterButtonClick () {
    //   this.getServer()
    //   this.addClusterDialogVisible = true
    // },
    // async getServer () {
    //   const id = this.$store.state.currentProjectId
    //   const { data: res } = await this.$http.get(
    //     'collect/getServer',
    //     { params: { project_id: id }, headers: { Authorization: this.$store.getters.getToken } }
    //   )
    //   this.serverList = res.data
    // },
    // async addCluster () {
    //   this.addClusterForm.project_id = this.$store.state.currentProjectId
    //   await this.$http.post(
    //     'collect/addCluster',
    //     this.addClusterForm,
    //     { headers: { Authorization: this.$store.getters.getToken } }
    //   )
    //   this.cancelAddClusterForm()
    //   await this.getNodes()
    // },
    // cancelAddClusterForm () {
    //   this.resetAddClusterForm()
    //   this.addClusterDialogVisible = false
    // },
    // resetAddClusterForm () {
    //   this.addClusterForm = {
    //     name: '',
    //     description: '',
    //     project_id: 0,
    //     serverList: []
    //   }
    //   this.$refs.addClusterFormRef.resetFields()
    //   this.$refs.addClusterTableRef.clearSelection()
    // },
    // 采集模块
    async getCollectFormDetail (id) {
      const { data: res } = await this.$http.get(
        'collect/getCollectFormDetail',
        { params: { id: id }, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.selectApplicationDetail = res.data.selectApplicationDetail
      this.addCollectForm.collectPackageList = res.data.collectPackageList
    },
    cancelAddCollectForm () {
      this.resetAddCollectForm()
      this.addCollectDialogVisible = false
    },
    resetAddCollectForm () {
      this.addCollectForm = {
        name: '',
        pattern: '',
        deploy_id: '请选择',
        is_default: true,
        log_name: '',
        use_case_name: '',
        project_id: 0,
        collectPackageList: []
      }
      this.$refs.addCollectFormRef.resetFields()
    },
    async addCollect () {
      this.addCollectForm.project_id = this.$store.state.currentProjectId
      await this.$http.post(
        'collect/addCollect',
        this.addCollectForm,
        { headers: { Authorization: this.$store.getters.getToken } }
      )
      this.cancelAddCollectForm()
      await this.getCollect()
    },
    async getCollect () {
      this.getCollectParams.project_id = this.$store.state.currentProjectId
      const { data: res } = await this.$http.get(
        'collect/getCollect',
        { params: this.getCollectParams, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.collectList = res.data
      if (this.collectList !== undefined && this.collectList !== null && this.collectList.length > 0) {
        this.collectAmount = this.collectList[0].collectAmount
      } else {
        this.collectAmount = 0
      }
      for (const item of this.collectList) {
        let created = item.collect.created
        created = created.substring(0, 10) + ' ' + created.substring(11)
        this.collectCreatedList.push(created)
      }
    },
    collectSizeChange (newPageSize) {
      this.getCollectParams.currentPage = 1
      this.getCollectParams.pageSize = newPageSize
      this.getCollect()
    },
    collectCurrentChange (newCurrentPage) {
      this.getCollectParams.currentPage = newCurrentPage
      this.getCollect()
    },
    async buildCollect (id) {
      await this.$http.post(
        'collect/buildCollect',
        { id: id },
        { headers: { Authorization: this.$store.getters.getToken } }
      )
      await this.getCollect()
    },
    async overCollect (id) {
      await this.$http.post(
        'collect/overCollect',
        { id: id },
        { headers: { Authorization: this.$store.getters.getToken } }
      )
      await this.getCollect()
    },
    async deleteCollect (index) {
    },
    openDeleteCollectBox (index) {
      Vue.prototype.$confirm('此操作将删除该采集任务, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.deleteCollect(index)
      }).catch(() => {
        Vue.prototype.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    handleCollectSelectionChange (selection) {
      this.collectSelected = selection
    },
    async CollectBatch () {
    },
    async deleteCollectBatch () {
    },
    async queryCollect () {
    },
    async downloadLog (id) {
      window.open(axios.defaults.baseURL + '/collect/downloadLog?id=' + id, '_blank')
    },
    async previewAgentConfig (id) {
      const { data: res } = await this.$http.get(
        'collect/previewAgentConfig',
        { params: { collect_id: id }, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.agentConfigDialogVisible = true
      this.agentConfigCode = res.data
      this.agentConfigCodeVisible = true
    },
    async previewConfig (id) {
      const { data: res } = await this.$http.get(
        'collect/previewConfig',
        { params: { collect_id: id }, headers: { Authorization: this.$store.getters.getToken } }
      )
      this.configDialogVisible = true
      this.configCode = res.data
      this.configCodeVisible = true
    }
  },
  computed: {
    // 部署模块
    uploadData: function () {
      return this.addDeployForms[0]
    },
    scriptServerComputedData: function () {
      return this.scriptServerData
    }
  },
  watch: {
    '$store.state.currentProjectId': function () {
      this.getNodes()
      this.getDeploy()
      this.getCollect()
    }
  }
}
</script>

<style scoped>
  .el-breadcrumb {
    margin-bottom: 15px;
  }

  .el-card {
    box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15) !important;
  }
</style>
<style scoped>
.dialogDiv {
  height: 50px;
  overflow: auto;
}
</style>
