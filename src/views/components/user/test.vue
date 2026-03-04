<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>可编辑表格示例</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/vue@3.3.4/dist/vue.global.prod.min.js"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            primary: '#3b82f6',
            secondary: '#64748b',
            success: '#10b981',
            danger: '#ef4444',
            warning: '#f59e0b',
            info: '#06b6d4',
            light: '#f8fafc',
            dark: '#1e293b',
          },
          fontFamily: {
            inter: ['Inter', 'sans-serif'],
          },
        },
      }
    }
  </script>
  <style type="text/tailwindcss">
    @layer utilities {
      .content-auto {
        content-visibility: auto;
      }
      .table-shadow {
        box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
      }
      .fade-in {
        animation: fadeIn 0.3s ease-in-out;
      }
      @keyframes fadeIn {
        from { opacity: 0; }
        to { opacity: 1; }
      }
    }
  </style>
</head>
<body class="bg-gray-50 font-inter text-gray-800 min-h-screen">
  <div id="app" class="container mx-auto px-4 py-8 max-w-6xl">
    <header class="mb-8">
      <h1 class="text-[clamp(1.5rem,3vw,2.5rem)] font-bold text-gray-800 mb-2">可编辑表格示例</h1>
      <p class="text-gray-600">双击姓名单元格进行编辑，编辑完成后将自动保存</p>
    </header>
    
    <div class="bg-white rounded-xl p-6 table-shadow mb-6 transition-all duration-300 hover:shadow-lg">
      <div class="overflow-x-auto">
        <table class="w-full">
          <thead>
            <tr class="border-b border-gray-200">
              <th class="text-left py-3 px-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">ID</th>
              <th class="text-left py-3 px-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">姓名</th>
              <th class="text-left py-3 px-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">年龄</th>
              <th class="text-left py-3 px-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">职位</th>
              <th class="text-left py-3 px-4 text-sm font-semibold text-gray-600 uppercase tracking-wider">操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in tableData" :key="item.id" class="border-b border-gray-100 hover:bg-gray-50 transition-colors duration-200">
              <td class="py-3 px-4 text-sm text-gray-600">{{ item.id }}</td>
              <td class="py-3 px-4">
                <div v-if="editingId !== item.id" class="cursor-pointer hover:text-primary transition-colors duration-200" @dblclick="startEditing(item.id, item.name)">
                  {{ item.name }}
                </div>
                <div v-else class="flex items-center space-x-2">
                  <input 
                    type="text" 
                    v-model="editedValue" 
                    class="w-full px-3 py-1 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary transition-all duration-200"
                    @blur="saveEditing(item.id)"
                    @keyup.enter="saveEditing(item.id)"
                    @keyup.escape="cancelEditing"
                    ref="editInput"
                  >
                  <button @click="saveEditing(item.id)" class="text-success hover:text-success/80 transition-colors">
                    <i class="fa fa-check"></i>
                  </button>
                  <button @click="cancelEditing" class="text-danger hover:text-danger/80 transition-colors">
                    <i class="fa fa-times"></i>
                  </button>
                </div>
              </td>
              <td class="py-3 px-4 text-sm text-gray-600">{{ item.age }}</td>
              <td class="py-3 px-4 text-sm text-gray-600">{{ item.position }}</td>
              <td class="py-3 px-4">
                <button class="text-primary hover:text-primary/80 transition-colors mr-3" @click="viewDetails(item.id)">
                  <i class="fa fa-eye"></i>
                </button>
                <button class="text-danger hover:text-danger/80 transition-colors" @click="deleteItem(item.id)">
                  <i class="fa fa-trash"></i>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      
      <!-- 加载状态 -->
      <div v-if="isLoading" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 fade-in">
        <div class="bg-white p-6 rounded-lg flex items-center space-x-4">
          <div class="animate-spin rounded-full h-10 w-10 border-t-2 border-b-2 border-primary"></div>
          <p class="text-gray-700">正在保存数据...</p>
        </div>
      </div>
      
      <!-- 操作提示 -->
      <div v-if="showMessage" class="fixed bottom-4 right-4 bg-success text-white px-6 py-3 rounded-lg shadow-lg flex items-center space-x-3 fade-in">
        <i class="fa fa-check-circle"></i>
        <p>{{ message }}</p>
      </div>
    </div>
  </div>

  <script>
    const { createApp, ref, reactive, onMounted, nextTick } = Vue;

    createApp({
      setup() {
        // 表格数据
        const tableData = reactive([
          { id: 1, name: '张三', age: 28, position: '前端开发' },
          { id: 2, name: '李四', age: 32, position: '后端开发' },
          { id: 3, name: '王五', age: 25, position: '产品经理' },
          { id: 4, name: '赵六', age: 29, position: 'UI设计师' },
          { id: 5, name: '钱七', age: 31, position: '测试工程师' }
        ]);
        
        // 当前编辑的ID
        const editingId = ref(null);
        // 编辑的值
        const editedValue = ref('');
        // 加载状态
        const isLoading = ref(false);
        // 消息提示
        const showMessage = ref(false);
        const message = ref('');
        // 编辑输入框引用
        const editInput = ref(null);

        // 开始编辑
        const startEditing = (id, value) => {
          editingId.value = id;
          editedValue.value = value;
          
          // 确保DOM更新后聚焦输入框
          nextTick(() => {
            editInput.value.focus();
            editInput.value.select();
          });
        };

        // 保存编辑
        const saveEditing = async (id) => {
          if (!editedValue.value.trim()) {
            showMessageHandler('姓名不能为空', 'error');
            return;
          }
          
          // 模拟API调用
          isLoading.value = true;
          
          try {
            // 调用后端API
            await updateNameApi(id, editedValue.value);
            
            // 更新本地数据
            const index = tableData.findIndex(item => item.id === id);
            if (index !== -1) {
              tableData[index].name = editedValue.value;
            }
            
            // 显示成功消息
            showMessageHandler('修改成功', 'success');
          } catch (error) {
            // 显示错误消息
            showMessageHandler('修改失败', 'error');
          } finally {
            // 重置状态
            editingId.value = null;
            isLoading.value = false;
          }
        };

        // 取消编辑
        const cancelEditing = () => {
          editingId.value = null;
        };

        // 查看详情（示例方法）
        const viewDetails = (id) => {
          alert(`查看ID为 ${id} 的详情`);
        };

        // 删除项（示例方法）
        const deleteItem = (id) => {
          if (confirm('确定要删除这项数据吗？')) {
            const index = tableData.findIndex(item => item.id === id);
            if (index !== -1) {
              tableData.splice(index, 1);
              showMessageHandler('删除成功', 'success');
            }
          }
        };

        // 显示消息提示
        const showMessageHandler = (msg, type = 'success') => {
          message.value = msg;
          showMessage.value = true;
          
          // 3秒后隐藏消息
          setTimeout(() => {
            showMessage.value = false;
          }, 3000);
        };

        // 模拟更新姓名的API调用
        const updateNameApi = (id, name) => {
          return new Promise((resolve, reject) => {
            // 模拟网络延迟
            setTimeout(() => {
              // 随机模拟成功或失败
              if (Math.random() > 0.1) {
                resolve();
              } else {
                reject(new Error('服务器错误'));
              }
            }, 800);
          });
        };

        return {
          tableData,
          editingId,
          editedValue,
          isLoading,
          showMessage,
          message,
          editInput,
          startEditing,
          saveEditing,
          cancelEditing,
          viewDetails,
          deleteItem
        };
      }
    }).mount('#app');
  </script>
</body>
</html>  