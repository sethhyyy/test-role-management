<template>
  <v-container fluid class="role-management">
    <div class="d-flex justify-space-between align-center mb-6">
      <h1 class="text-h4 font-weight-bold text-primary-dark">จัดการสิทธิ์ผู้ใช้งาน</h1>
      <v-btn color="primary" dark @click="openAddModal" class="text-none px-6" style="height: 40px;">
        <v-icon left>mdi-plus</v-icon>
        เพิ่ม Role
      </v-btn>
    </div>

    <!-- Users Table -->
    <v-card class="mb-4 table-card" elevation="0" outlined>
      <v-data-table
        :headers="headers"
        :items="filteredUsers"
        item-key="id"
        class="elevation-0 user-table"
        :class="{ 'has-deleted': showDeleted }"
        hide-default-footer
      >
        <template #[`item.no`]="{ item }">
          <span class="text-medium">{{ item.id }}</span>
        </template>

        <template #[`item.fullname`]="{ item }">
          <span class="font-weight-medium">{{ item.fullname }}</span>
        </template>
        
        <template #[`item.id_card`]="{ item }">
          <span>{{ item.id_card || '-' }}</span>
        </template>
        
        <template #[`item.phone`]="{ item }">
          <span>{{ item.phone || '-' }}</span>
        </template>
        
        <template #[`item.email`]="{ item }">
          <span>{{ item.email }}</span>
        </template>
        
        <template #[`item.role`]="{ item }">
          <v-chip
            :color="item.role === 'Super Admin' ? '#ffcdd2' : '#c8e6c9'"
            :text-color="item.role === 'Super Admin' ? '#c62828' : '#2e7d32'"
            small
            label
            class="role-chip px-4"
          >
            {{ item.role }}
          </v-chip>
        </template>
        
        <template #[`item.actions`]="{ item }">
          <div class="d-flex">
            <v-btn
              icon
              small
              color="#1976d2"
              @click="openEditModal(item)"
              :disabled="item.isDeleted"
              class="action-btn"
            >
              <v-icon small>mdi-pencil</v-icon>
            </v-btn>
            <v-btn
              icon
              small
              color="#d32f2f"
              @click="deleteUser(item)"
              :disabled="item.isDeleted"
              class="action-btn"
            >
              <v-icon small>mdi-delete</v-icon>
            </v-btn>
          </div>
        </template>
      </v-data-table>
    </v-card>

    <!-- Show Deleted Toggle -->
    <v-card elevation="0" outlined class="py-2 px-3" style="background-color: #f8f9fa;">
      <v-checkbox
        v-model="showDeleted"
        label="แสดงผู้ใช้ที่ถูกลบแล้ว"
        hide-details
        class="mt-0"
        dense
      ></v-checkbox>
    </v-card>

    <!-- Add/Edit Dialog -->
    <v-dialog v-model="showModal" max-width="600px" persistent>
      <v-card class="dialog-card">
        <v-card-title class="dialog-header d-flex align-center">
          <span class="text-h6 font-weight-bold white--text">{{ isEditMode ? 'แก้ไขผู้ใช้' : 'เพิ่มผู้ใช้ใหม่' }}</span>
          <v-spacer></v-spacer>
          <v-btn icon dark @click="closeModal">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        
        <v-card-text class="pt-6">
          <v-form ref="form" v-model="valid" lazy-validation>
            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="formData.firstname"
                  label="ชื่อ"
                  outlined
                  dense
                  :rules="[v => !!v || 'กรุณากรอกชื่อ']"
                  class="mb-2"
                ></v-text-field>
              </v-col>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="formData.lastname"
                  label="นามสกุล"
                  outlined
                  dense
                  :rules="[v => !!v || 'กรุณากรอกนามสกุล']"
                  class="mb-2"
                ></v-text-field>
              </v-col>
            </v-row>
            
            <v-text-field
              v-model="formData.id_card"
              label="เลขบัตรประชาชน"
              outlined
              dense
              :rules="[v => !v || /^\d{13}$/.test(v) || 'กรุณากรอกเลขบัตร 13 หลัก']"
              maxlength="13"
              class="mb-2"
            ></v-text-field>
            
            <v-text-field
              v-model="formData.phone"
              label="เบอร์โทร"
              outlined
              dense
              :rules="[v => !v || /^0\d{9}$/.test(v) || 'กรุณากรอกเบอร์โทร 10 หลัก']"
              maxlength="10"
              class="mb-2"
            ></v-text-field>
            
            <v-text-field
              v-model="formData.email"
              label="อีเมล"
              outlined
              dense
              :rules="[v => !!v || 'กรุณากรอกอีเมล', v => /.+@.+\..+/.test(v) || 'อีเมลไม่ถูกต้อง']"
              class="mb-4"
            ></v-text-field>
            
            <v-select
              v-model="formData.role"
              :items="roleItems"
              label="ตำแหน่ง"
              outlined
              dense
              :rules="[v => !!v || 'กรุณาเลือกตำแหน่ง']"
              class="mb-4"
            ></v-select>
          </v-form>
        </v-card-text>
        
        <v-card-actions class="pa-4">
          <v-spacer></v-spacer>
          <v-btn color="grey" text @click="closeModal" class="mr-2">ยกเลิก</v-btn>
          <v-btn color="primary" dark @click="saveUser">{{ isEditMode ? 'บันทึก' : 'เพิ่มผู้ใช้' }}</v-btn>
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
          <p>คุณต้องการลบผู้ใช้ <strong>{{ userToDelete?.fullname }}</strong> ใช่หรือไม่?</p>
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
      users: [],
      showDeleted: false,
      showModal: false,
      showDeleteModal: false,
      isEditMode: false,
      userToDelete: null,
      formData: {
        id: null,
        firstname: '',
        lastname: '',
        id_card: '',
        phone: '',
        email: '',
        role: 'Admin',
        isDeleted: false
      },
      nextId: 9,
      headers: [
        { text: 'ลำดับ', value: 'no', width: 80 },
        { text: 'ชื่อ-นามสกุล', value: 'fullname', width: 200 },
        { text: 'เลขบัตรประชาชน', value: 'id_card', width: 150 },
        { text: 'เบอร์โทร', value: 'phone', width: 130 },
        { text: 'อีเมล', value: 'email', width: 180 },
        { text: 'ตำแหน่ง', value: 'role', width: 120 },
        { text: 'จัดการ', value: 'actions', width: 100, sortable: false }
      ],
      roleItems: ['Super Admin', 'Admin', 'User']
    }
  },
  computed: {
    filteredUsers() {
      if (this.showDeleted) {
        return this.users
      }
      return this.users.filter(user => !user.isDeleted)
    }
  },
  created() {
    this.loadMockData()
  },
  methods: {
    loadMockData() {
      this.users = [
        {
          id: 1,
          firstname: 'พิภัทร',
          lastname: 'วรปาณิ',
          fullname: 'นายพิภัทร วรปาณิ',
          id_card: '1659902026799',
          phone: '0823555595',
          email: 'packaq2@gmail.com',
          role: 'Super Admin',
          isDeleted: false
        },
        {
          id: 2,
          firstname: 'ชื่อ',
          lastname: 'นามสกุล',
          fullname: 'นายชื่อ นามสกุล',
          id_card: '',
          phone: '123',
          email: 'test@email.com',
          role: 'Admin',
          isDeleted: false
        },
        {
          id: 3,
          firstname: 'ชื่อ',
          lastname: 'นามสกุล',
          fullname: 'นายชื่อ นามสกุล',
          id_card: '',
          phone: '123',
          email: 'test@email.com',
          role: 'Admin',
          isDeleted: false
        },
        {
          id: 4,
          firstname: 'ชื่อ2',
          lastname: 'นามสกุล2',
          fullname: 'นายชื่อ2 นามสกุล2',
          id_card: '123',
          phone: '123456',
          email: '2test@email.com',
          role: 'Super Admin',
          isDeleted: false
        },
        {
          id: 5,
          firstname: 'ชื่อ',
          lastname: 'นามสกุล',
          fullname: 'นายชื่อ นามสกุล',
          id_card: '',
          phone: '123',
          email: 'test@email.com',
          role: 'Admin',
          isDeleted: false
        },
        {
          id: 6,
          firstname: 'แอดมิน',
          lastname: 'ระบบ',
          fullname: 'นายแอดมิน ระบบ',
          id_card: '',
          phone: '0000000000',
          email: 'test@email.com',
          role: 'Super Admin',
          isDeleted: false
        },
        {
          id: 7,
          firstname: 'ทดสอบหนึ่ง',
          lastname: 'ทดสอบหนึ่ง',
          fullname: 'นางสาวทดสอบหนึ่ง ทดสอบหนึ่ง',
          id_card: '1208804989536',
          phone: '0855855858',
          email: 'test@testmail.com',
          role: 'Admin',
          isDeleted: false
        },
        {
          id: 8,
          firstname: 'ทดสอบสอง',
          lastname: 'ทดสอบสอง',
          fullname: 'นางทดสอบสอง ทดสอบสอง',
          id_card: '7073577320823',
          phone: '0874747747',
          email: 'test@testmail.com',
          role: 'Admin',
          isDeleted: false
        }
      ]
      this.nextId = this.users.length + 1
    },

    openAddModal() {
      this.isEditMode = false
      this.formData = {
        id: null,
        firstname: '',
        lastname: '',
        id_card: '',
        phone: '',
        email: '',
        role: 'Admin',
        isDeleted: false
      }
      this.showModal = true
      if (this.$refs.form) {
        this.$refs.form.resetValidation()
      }
    },

    openEditModal(user) {
      this.isEditMode = true
      this.formData = {
        id: user.id,
        firstname: user.firstname || '',
        lastname: user.lastname || '',
        id_card: user.id_card || '',
        phone: user.phone || '',
        email: user.email,
        role: user.role,
        isDeleted: user.isDeleted
      }
      this.showModal = true
    },

    closeModal() {
      this.showModal = false
      this.formData = {}
    },

    openDeleteModal(user) {
      this.userToDelete = user
      this.showDeleteModal = true
    },

    closeDeleteModal() {
      this.showDeleteModal = false
      this.userToDelete = null
    },

    deleteUser(user) {
      this.openDeleteModal(user)
    },

    confirmDelete() {
      if (this.userToDelete) {
        const index = this.users.findIndex(u => u.id === this.userToDelete.id)
        if (index !== -1) {
          this.users[index].isDeleted = true
        }
      }
      this.closeDeleteModal()
    },

    saveUser() {
      if (this.$refs.form && !this.$refs.form.validate()) {
        return
      }

      // fullname จะถูกสร้างจาก firstname และ lastname ในบรรทัดด้านล่าง

      if (this.isEditMode) {
        const index = this.users.findIndex(u => u.id === this.formData.id)
        if (index !== -1) {
          this.users[index] = {
            ...this.users[index],
            firstname: this.formData.firstname,
            lastname: this.formData.lastname,
            fullname: `${this.formData.firstname} ${this.formData.lastname}`,
            id_card: this.formData.id_card,
            phone: this.formData.phone,
            email: this.formData.email,
            role: this.formData.role
          }
        }
      } else {
        const newUser = {
          id: this.nextId++,
          firstname: this.formData.firstname,
          lastname: this.formData.lastname,
          fullname: `${this.formData.firstname} ${this.formData.lastname}`,
          id_card: this.formData.id_card,
          phone: this.formData.phone,
          email: this.formData.email,
          role: this.formData.role,
          isDeleted: false
        }
        this.users.push(newUser)
      }
      this.closeModal()
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
  background-color: #ffffff !important;
}

.user-table >>> thead th {
  background-color: #e3f2fd !important;
  color: #1565c0 !important;
  font-weight: 600 !important;
  font-size: 0.9rem;
  border: none !important;
  padding: 16px 12px !important;
  text-align: center;
}

.user-table >>> thead th:first-child {
  border-radius: 8px 0 0 0;
}

.user-table >>> thead th:last-child {
  border-radius: 0 8px 0 0;
}

.user-table >>> tbody tr {
  border-bottom: 1px solid #f0f0f0;
}

.user-table >>> tbody tr:hover {
  background-color: #f5f9ff !important;
}

.user-table >>> tbody td {
  padding: 14px 12px !important;
  font-size: 0.9rem;
  color: #333;
  text-align: center;
}

.user-table >>> tbody td:first-child {
  text-align: left;
}

.role-chip {
  font-weight: 500;
  font-size: 0.85rem;
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