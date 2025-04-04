<template>
  <div class="container">
    <!-- ข้อมูลส่วนตัว -->
    <div class="card mt-4 card-custom">
      <div class="card-body d-flex">
        <img src="../public/img/nok.jpg" alt="รูปภาพของฉัน" class="img-thumbnail mb-3" style="width: 300px; height: 400px; object-fit: cover;" />
        <div class="card-body d-flex align-items-center">
        <div class="me-5">
          <h5 class="card-title">ข้อมูลส่วนตัว</h5>

    <p class="text-start w-100">
</p>
    <p><strong>รหัสนิสิต:</strong> {{ student.id }}</p>
    <p><strong>👤ชื่อ:</strong> {{ student.name }}</p>
    <p><strong>🏷️รหัสสาขา:</strong> {{ student.major_id }}</p>
    <p><strong>📖สาขา:</strong> {{ student.major_name }}</p>
    <p><strong>🏫โรงเรียนเดิม:</strong> {{ student.old_school }}</p>

    <button class="btn btn-primary" @click="toggleProfileEdit">แก้ไขข้อมูลส่วนตัว</button>
        </div>
  </div>
  
        <img src="../public/img/bird.jpg" alt="รูปภาพของฉัน" 
     class="img-thumbnail mb-3 ms-auto bird" 
     style="width: 300px; height: 400px; object-fit: cover;" />


      </div>
    </div>

    <div v-if="isProfileEditMode" class="mt-3">
            <h5>แก้ไขข้อมูลส่วนตัว</h5>
            <form @submit.prevent="saveStudent">
              <div class="mb-3">
                <label for="name" class="form-label">ชื่อ</label>
                <input v-model="editStudent.name" type="text" id="name" class="form-control" required />
              </div>
              <div class="mb-3">
                <label for="major_id" class="form-label">รหัสสาขา</label>
                <input v-model="editStudent.major_id" type="text" id="major_id" class="form-control" required />
              </div>
              <div class="mb-3">
                <label for="major_name" class="form-label">สาขา</label>
                <input v-model="editStudent.major_name" type="text" id="major_name" class="form-control" required />
              </div>
              <div class="mb-3">
                <label for="old_school" class="form-label">โรงเรียนเดิม</label>
                <input v-model="editStudent.old_school" type="text" id="old_school" class="form-control" required />
              </div>
              <button type="submit" class="btn btn-success">บันทึก</button>
            </form>
          </div>

    <!-- รายการวิชา -->
    <div class="card mt-4 card-custom">
      <div class="card-body">
        <h5 class="card-title">รายการวิชา</h5>
        <table class="table">
          <thead>
            <tr>
              <th>ชื่อวิชา</th>
              <th>รหัสวิชา</th>
              <th>เกรด</th>
              <th>หน่วยกิต</th>
              <th>การจัดการ</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(subject, index) in subjects" :key="subject.id">
              <td>{{ subject.name }}</td>
              <td>{{ subject.id }}</td>
              <td>{{ subject.grade }}</td>
              <td>{{ subject.credit }}</td>
              <td>
                <button class="btn btn-info btn-sm" @click="openModal('edit', subject)">แก้ไข</button>
                <button class="btn btn-danger btn-sm" @click="deleteSubject(index)">ลบ</button>
              </td>
            </tr>
          </tbody>
        </table>
        <button class="btn btn-primary" @click="openModal('add')">เพิ่มวิชา</button>
      </div>
    </div>

    <!-- Modal สำหรับเพิ่มและแก้ไขวิชา -->
    <div class="modal fade" id="subjectModal" tabindex="-1" aria-labelledby="subjectModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="subjectModalLabel">{{ isEditMode ? 'แก้ไขวิชา' : 'เพิ่มวิชา' }}</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="saveSubjectForm">
              <div class="mb-3">
                <label for="subject-name" class="form-label">ชื่อวิชา</label>
                <input v-model="currentSubject.name" type="text" id="subject-name" class="form-control" required />
              </div>
              <div class="mb-3">
                <label for="subject-id" class="form-label">รหัสวิชา</label>
                <input v-model="currentSubject.id" type="text" id="subject-id" class="form-control" required />
              </div>
              <div class="mb-3">
                <label for="subject-grade" class="form-label">เกรด</label>
                <input v-model="currentSubject.grade" type="text" id="subject-grade" class="form-control" required />
              </div>
              <div class="mb-3">
                <label for="subject-credit" class="form-label">หน่วยกิต</label>
                <input v-model="currentSubject.credit" type="text" id="subject-credit" class="form-control" required />
              </div>
              <button type="submit" class="btn btn-success">{{ isEditMode ? 'บันทึก' : 'เพิ่ม' }}</button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

let id_now = "";

// กำหนด URL ของ json-server
const apiUrl = 'http://localhost:3000';

// สร้างตัวแปรสำหรับข้อมูลส่วนตัว
const student = ref({
  id: '',
  name: '',
  major_id: '',
  major_name: '',
  old_school: '',
});


const editStudent = ref({})

// สร้างตัวแปรสำหรับรายการวิชา
const subjects = ref([]);
const isProfileEditMode = ref(false);

// สร้างตัวแปรสำหรับการแสดงฟอร์มแก้ไข
const isEditMode = ref(false);
const currentSubject = ref({});

const saveStudent = async () => {
  try {
    await fetch(`${apiUrl}/students/`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(editStudent.value),
    });
    student.value = { ...editStudent.value };
    isProfileEditMode.value = false; // ปิดโหมดแก้ไข
  } catch (error) {
    console.error('Error saving student data:', error);
  }
};


const toggleProfileEdit = () => {
  isProfileEditMode.value = !isProfileEditMode.value;
  editStudent.value = { ...student.value };
};

// ฟังก์ชันโหลดข้อมูลจาก json-server
const loadData = async () => {
  try {
    // โหลดข้อมูลนักศึกษา
    const studentResponse = await fetch(`${apiUrl}/students/`); // ใช้ id 1 ตามตัวอย่าง
    const studentData = await studentResponse.json();
    student.value = studentData;

    // โหลดข้อมูลวิชา
    const subjectsResponse = await fetch(`${apiUrl}/subjects`);
    const subjectsData = await subjectsResponse.json();
    subjects.value = subjectsData;
  } catch (error) {
    console.error('Error loading data:', error);
  }
};

// ฟังก์ชันเปิด Modal สำหรับเพิ่มหรือแก้ไขวิชา
const openModal = (mode, subject = {}) => {
  isEditMode.value = mode === 'edit';
  currentSubject.value = { ...subject };
  id_now = subject.id;
  const modal = new bootstrap.Modal(document.getElementById('subjectModal'));
  modal.show();
};

// ฟังก์ชันบันทึกข้อมูลวิชา
const saveSubjectForm = async () => {
  try {
    // ถ้าอยู่ในโหมดแก้ไข
    if (isEditMode.value) {
      // เช็คว่ารหัสวิชาของ currentSubject มีการเปลี่ยนแปลงหรือไม่
      if (currentSubject.value.id !== id_now.value) {
        // ลบวิชาเก่าออกก่อน
        await fetch(`${apiUrl}/subjects/${id_now}`, {
          method: 'DELETE',
          headers: { 'Content-Type': 'application/json' },
        });
      }

      // เพิ่มวิชาใหม่ (หรือแก้ไขข้อมูลเดิม)
      await fetch(`${apiUrl}/subjects/`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(currentSubject.value),
      });
    } else {
      // สำหรับการเพิ่มวิชาใหม่
      await fetch(`${apiUrl}/subjects`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(currentSubject.value),
      });
    }

    // รีโหลดข้อมูลหลังจากบันทึก
    loadData();

    // ปิด Modal
    const modal = bootstrap.Modal.getInstance(document.getElementById('subjectModal'));
    modal.hide();
  } catch (error) {
    console.error('Error saving subject:', error);
  }
};


// ฟังก์ชันลบวิชา
const deleteSubject = async (index) => {
  const subjectId = subjects.value[index].id;

  try {
    await fetch(`${apiUrl}/subjects/${subjectId}`, {
      method: 'DELETE',
    });
    subjects.value.splice(index, 1);
  } catch (error) {
    console.error('Error deleting subject:', error);
  }
};

// เรียกใช้งานเมื่อคอมโพเนนต์ถูกติดตั้ง
onMounted(loadData);
</script>

<style scoped>
.container {
  background-color: #66ff736b;
}

.card-custom {
  border: 2px solid #007bff;
  border-radius: 8px;
  background-color: #80d6f8;
}
.bird{
  margin-right: 70px;
}
</style>
