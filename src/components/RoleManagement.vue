<template>
  <v-container fluid class="role-management">
    <div class="d-flex justify-space-between align-center mb-6">
      <h1 class="text-h4 font-weight-bold text-primary-dark">จัดการสิทธิ์ผู้ใช้งาน</h1>
      <v-btn color="primary" dark @click="openAddModal" class="text-none px-6" style="height: 40px;">
        <v-icon left>mdi-plus</v-icon>
        เพิ่ม Role
      </v-btn>
    </div>

    <!-- Roles Table -->
    <v-card class="mb-4 table-card" elevation="0" outlined>
      <v-data-table
        :headers="headers"
        :items="filteredRoles"
        item-key="id"
        class="elevation-0 role-table"
        :class="{ 'has-deleted': showDeleted }"
        hide-default-footer
      >
        <template #[`item.no`]="{ item }">
          <span class="text-medium">{{ item.id }}</span>
        </template>

        <template #[`item.role_name`]="{ item }">
          <span class="font-weight-medium">{{ item.role_name }}</span>
        </template>
        
        <template #[`item.role_level`]="{ item }">
          <v-chip
            :color="getRoleLevelColor(item.role_level)"
            text-color="white"
            small
            label
            class="role-chip"
          >
            {{ getRoleLevelText(item.role_level) }}
          </v-chip>
        </template>
        
        <template #[`item.tab_allowed`]="{ item }">
          <div class="d-flex flex-wrap" style="gap: 4px;">
            <v-chip
              v-for="tabKey in getMainTabs(item.tab_allowed)"
              :key="tabKey"
              color="success"
              text-color="white"
              x-small
              label
            >
              {{ getTabLabel(tabKey) }}
            </v-chip>
          </div>
        </template>
        
        <template #[`item.deletable`]="{ item }">
          <v-chip
            :color="item.deletable ? 'success' : 'error'"
            text-color="white"
            x-small
            label
          >
            {{ item.deletable ? 'ใช่' : 'ไม่' }}
          </v-chip>
        </template>
        
        <template #[`item.create_at`]="{ item }">
          <span class="text-grey">{{ formatDate(item.create_at) }}</span>
        </template>
        
        <template #[`item.update_at`]="{ item }">
          <span class="text-grey">{{ formatDate(item.update_at) }}</span>
        </template>
        
        <template #[`item.actions`]="{ item }">
          <v-btn
            icon
            small
            color="primary"
            @click="openEditModal(item)"
            :disabled="item.isDeleted"
            class="action-btn"
          >
            <v-icon small>mdi-pencil</v-icon>
          </v-btn>
          <v-btn
            icon
            small
            color="error"
            @click="deleteRole(item)"
            :disabled="!item.deletable || item.isDeleted"
            class="action-btn"
          >
            <v-icon small>mdi-delete</v-icon>
          </v-btn>
        </template>
        
        <template #[`item.data-table-expand`]="{ item }">
          <span v-if="item.isDeleted" class="text-caption grey--text">ถูกลบแล้ว</span>
        </template>
      </v-data-table>
    </v-card>

    <!-- Show Deleted Toggle -->
    <v-card elevation="0" outlined class="py-2 px-3" style="background-color: #f8f9fa;">
      <v-checkbox
        v-model="showDeleted"
        label="แสดง Role ที่ถูกลบแล้ว"
        hide-details
        class="mt-0"
        dense
      ></v-checkbox>
    </v-card>

    <!-- Add/Edit Dialog -->
    <v-dialog v-model="showModal" max-width="600px" persistent>
      <v-card class="dialog-card">
        <v-card-title class="dialog-header d-flex align-center">
          <span class="text-h6 font-weight-bold white--text">{{ isEditMode ? 'แก้ไข Role' : 'เพิ่ม Role ใหม่' }}</span>
          <v-spacer></v-spacer>
          <v-btn icon dark @click="closeModal">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        
        <v-card-text class="pt-6">
          <v-form ref="form" v-model="valid" lazy-validation>
            <v-text-field
              v-model="formData.role_name"
              label="ชื่อ Role"
              outlined
              dense
              :rules="[v => !!v || 'กรุณากรอกชื่อ Role']"
              :disabled="isEditMode"
              class="mb-4"
            ></v-text-field>
            
            <v-select
              v-model="formData.role_level"
              :items="roleLevelItems"
              label="ระดับ Role"
              outlined
              dense
              :rules="[v => v !== null && v !== undefined || 'กรุณาเลือกระดับ Role']"
              :disabled="isEditMode"
              class="mb-4"
            ></v-select>
            
            <div class="mb-4">
              <label class="text-subtitle-2 font-weight-medium mb-2 d-block">แท็บที่อนุญาต</label>
              <v-card outlined class="pa-3" style="max-height: 300px; overflow-y: auto; background-color: #fafafa;">
                <v-row dense>
                  <v-col
                    v-for="tab in availableTabs"
                    :key="tab.key"
                    cols="6"
                    :class="{ 'pl-8': tab.parent }"
                  >
                    <v-checkbox
                      v-model="selectedTabs"
                      :value="tab.key"
                      :label="tab.label"
                      hide-details
                      dense
                      class="mt-0"
                      :disabled="tab.parent && !selectedTabs.includes(tab.parent)"
                    ></v-checkbox>
                  </v-col>
                </v-row>
              </v-card>
            </div>
            
            <v-checkbox
              v-model="formData.deletable"
              label="สามารถลบได้ (Deletable)"
              hide-details
              dense
              class="mt-0"
            ></v-checkbox>
          </v-form>
        </v-card-text>
        
        <v-card-actions class="pa-4">
          <v-spacer></v-spacer>
          <v-btn color="grey" text @click="closeModal" class="mr-2">ยกเลิก</v-btn>
          <v-btn color="primary" dark @click="saveRole">{{ isEditMode ? 'บันทึก' : 'เพิ่ม Role' }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Delete Confirmation Dialog -->
    <v-dialog v-model="showDeleteModal" max-width="400px" persistent>
      <v-card>
        <v-card-title class="dialog-header-error d-flex align-center">
          <v-icon left>mdi-alert</v-icon>
          <span class="text-h6 font-weight-bold white--text">ยืนยันการลบ</span>
        </v-card-title>
        <v-card-text class="pt-6">
          <p>คุณต้องการลบ Role <strong>{{ roleToDelete?.role_name }}</strong> ใช่หรือไม่?</p>
          <v-alert type="warning" dense text class="mt-3">
            การกระทำนี้ไม่สามารถย้อนกลับได้
          </v-alert>
        </v-card-text>
        <v-card-actions class="pa-4">
          <v-spacer></v-spacer>
          <v-btn color="grey" text @click="closeDeleteModal" class="mr-2">ยกเลิก</v-btn>
          <v-btn color="error" dark @click="confirmDelete">ลบ</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  name: 'RoleManagement',
  data() {
    return {
      valid: true,
      roles: [],
      showDeleted: false,
      showModal: false,
      showDeleteModal: false,
      isEditMode: false,
      roleToDelete: null,
      availableTabs: [
        { key: 'member_info', label: 'ข้อมูลสมาชิก' },
        { key: 'registration_docs', label: 'หลักฐานขึ้นทะเบียน' },
        { key: 'registration_docs_verify', label: 'ตรวจสอบหลักฐาน', parent: 'registration_docs' },
        { key: 'registration_docs_pending', label: 'รอขึ้นทะเบียน', parent: 'registration_docs' },
        { key: 'license_renewal_verify', label: 'ตรวจสอบหลักฐานต่อใบอนุญาต' },
        { key: 'license_renewal_pending', label: 'รอขึ้นทะเบียนใบอนุญาต', parent: 'license_renewal_verify' },
        { key: 'system_overview', label: 'ภาพรวมระบบ' },
        { key: 'edit_request', label: 'ขอแก้ไขข้อมูล' },
        { key: 'admin_management', label: 'จัดการผู้ดูแล' }
      ],
      selectedTabs: [],
      formData: {
        id: null,
        role_name: '',
        role_level: 0,
        tab_allowed: {},
        deletable: true,
        isDeleted: false
      },
      nextId: 4,
      headers: [
        { text: 'ลำดับ', value: 'no', width: 60 },
        { text: 'ชื่อ Role', value: 'role_name', width: 150 },
        { text: 'ระดับ', value: 'role_level', width: 100 },
        { text: 'แท็บที่อนุญาต', value: 'tab_allowed' },
        { text: 'ลบได้', value: 'deletable', width: 70 },
        { text: 'สร้างเมื่อ', value: 'create_at', width: 150 },
        { text: 'แก้ไขเมื่อ', value: 'update_at', width: 150 },
        { text: 'จัดการ', value: 'actions', width: 100, sortable: false }
      ],
      roleLevelItems: [
        { text: 'User (0)', value: 0 },
        { text: 'Admin (1)', value: 1 },
        { text: 'Superadmin (2)', value: 2 }
      ]
    }
  },
  computed: {
    filteredRoles() {
      if (this.showDeleted) {
        return this.roles
      }
      return this.roles.filter(role => !role.isDeleted)
    }
  },
  created() {
    this.loadMockData()
  },
  methods: {
    loadMockData() {
      this.roles = [
        {
          id: 1,
          role_name: 'User',
          role_level: 0,
          tab_allowed: {
            member_info: true,
            registration_docs: false,
            registration_docs_verify: false,
            registration_docs_pending: false,
            license_renewal_verify: false,
            license_renewal_pending: false,
            system_overview: false,
            edit_request: true,
            admin_management: false
          },
          deletable: true,
          isDeleted: false,
          create_at: '2025-01-01T08:00:00Z',
          update_at: '2025-01-01T08:00:00Z'
        },
        {
          id: 2,
          role_name: 'Admin',
          role_level: 1,
          tab_allowed: {
            member_info: true,
            registration_docs: true,
            registration_docs_verify: true,
            registration_docs_pending: true,
            license_renewal_verify: true,
            license_renewal_pending: true,
            system_overview: false,
            edit_request: true,
            admin_management: false
          },
          deletable: false,
          isDeleted: false,
          create_at: '2025-01-01T08:00:00Z',
          update_at: '2025-01-01T08:00:00Z'
        },
        {
          id: 3,
          role_name: 'Superadmin',
          role_level: 2,
          tab_allowed: {
            member_info: true,
            registration_docs: true,
            registration_docs_verify: true,
            registration_docs_pending: true,
            license_renewal_verify: true,
            license_renewal_pending: true,
            system_overview: true,
            edit_request: true,
            admin_management: true
          },
          deletable: false,
          isDeleted: false,
          create_at: '2025-01-01T08:00:00Z',
          update_at: '2025-01-01T08:00:00Z'
        }
      ]
      this.nextId = this.roles.length + 1
    },

    getRoleLevelText(level) {
      const levels = {
        0: 'User',
        1: 'Admin',
        2: 'Superadmin'
      }
      return levels[level] || 'Unknown'
    },

    getRoleLevelColor(level) {
      const colors = {
        0: '#5c6bc0',
        1: '#66bb6a',
        2: '#ef5350'
      }
      return colors[level] || '#757575'
    },

    formatDate(dateString) {
      if (!dateString) return '-'
      const date = new Date(dateString)
      return date.toLocaleString('th-TH', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit'
      })
    },

    openAddModal() {
      this.isEditMode = false
      this.formData = {
        id: null,
        role_name: '',
        role_level: 0,
        tab_allowed: {},
        deletable: true,
        isDeleted: false
      }
      this.selectedTabs = []
      this.showModal = true
      if (this.$refs.form) {
        this.$refs.form.resetValidation()
      }
    },

    openEditModal(role) {
      this.isEditMode = true
      this.formData = {
        id: role.id,
        role_name: role.role_name,
        role_level: role.role_level,
        tab_allowed: { ...role.tab_allowed },
        deletable: role.deletable,
        isDeleted: role.isDeleted
      }
      this.selectedTabs = Object.keys(role.tab_allowed).filter(tab => role.tab_allowed[tab])
      this.showModal = true
    },

    closeModal() {
      this.showModal = false
      this.formData = {}
      this.selectedTabs = []
    },

    openDeleteModal(role) {
      this.roleToDelete = role
      this.showDeleteModal = true
    },

    closeDeleteModal() {
      this.showDeleteModal = false
      this.roleToDelete = null
    },

    deleteRole(role) {
      this.openDeleteModal(role)
    },

    confirmDelete() {
      if (this.roleToDelete) {
        const index = this.roles.findIndex(r => r.id === this.roleToDelete.id)
        if (index !== -1) {
          this.roles[index].isDeleted = true
        }
      }
      this.closeDeleteModal()
    },

    saveRole() {
      if (this.$refs.form && !this.$refs.form.validate()) {
        return
      }

      const tab_allowed = {}
      this.availableTabs.forEach(tab => {
        tab_allowed[tab.key] = this.selectedTabs.includes(tab.key)
      })

      if (this.isEditMode) {
        const index = this.roles.findIndex(r => r.id === this.formData.id)
        if (index !== -1) {
          this.roles[index] = {
            ...this.roles[index],
            tab_allowed,
            deletable: this.formData.deletable,
            update_at: new Date().toISOString()
          }
        }
      } else {
        const newRole = {
          id: this.nextId++,
          role_name: this.formData.role_name,
          role_level: parseInt(this.formData.role_level),
          tab_allowed,
          deletable: this.formData.deletable,
          isDeleted: false,
          create_at: new Date().toISOString(),
          update_at: new Date().toISOString()
        }
        this.roles.push(newRole)
      }
      this.closeModal()
    },

    getTabLabel(key) {
      const tab = this.availableTabs.find(t => t.key === key)
      return tab ? tab.label : key
    },

    isSubTab(key) {
      const tab = this.availableTabs.find(t => t.key === key)
      return tab && tab.parent
    },

    getMainTabs(tabAllowed) {
      return Object.keys(tabAllowed).filter(key => {
        const tab = this.availableTabs.find(t => t.key === key)
        return tab && !tab.parent && tabAllowed[key]
      })
    }
  }
}
</script>

<style scoped>
.role-management {
  background-color: #f5f8fa;
  min-height: 100vh;
  padding-top: 32px !important;
  padding-bottom: 32px !important;
}

.text-primary-dark {
  color: #1a237e;
}

.table-card {
  border-radius: 12px !important;
  overflow: hidden;
  border: 1px solid #e0e0e0 !important;
}

.role-table >>> thead th {
  background-color: #e3f2fd !important;
  color: #1565c0 !important;
  font-weight: 600 !important;
  font-size: 0.9rem;
  border: none !important;
  padding: 16px 12px !important;
}

.role-table >>> thead th:first-child {
  border-radius: 8px 0 0 0;
}

.role-table >>> thead th:last-child {
  border-radius: 0 8px 0 0;
}

.role-table >>> tbody tr {
  border-bottom: 1px solid #f0f0f0;
}

.role-table >>> tbody tr:hover {
  background-color: #f5f9ff !important;
}

.role-table >>> tbody td {
  padding: 14px 12px !important;
  font-size: 0.9rem;
  color: #333;
}

.role-chip {
  font-weight: 500;
  font-size: 0.8rem;
}

.action-btn {
  margin: 0 2px;
}

.has-deleted >>> tbody tr.row-deleted {
  background-color: #ffebee !important;
  opacity: 0.7;
}

.dialog-card {
  border-radius: 12px !important;
  overflow: hidden;
}

.dialog-header {
  background: linear-gradient(135deg, #1976d2 0%, #1565c0 100%);
  padding: 20px 24px;
}

.dialog-header-error {
  background: linear-gradient(135deg, #d32f2f 0%, #c62828 100%);
  padding: 20px 24px;
}

.text-grey {
  color: #757575;
}

.text-medium {
  font-weight: 500;
}
</style>