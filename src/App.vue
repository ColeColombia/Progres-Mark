<script setup>
import { onMounted, onUpdated, ref } from 'vue'

const subjectContents = ref([])
const outPut = ref([])
const subjectData = ref({subject:'', progressMark: '', markLeft:'', passMark:''})
const updatedSUbjectData = ref({subject:'', progressMark: '', markLeft:'', passMark:''})
const error = ref({errorMessage:'please fill in all the required details', error: false})
const showUpdateMenu = ref(false)
const showReset = ref(false)
const subjectUpdateIndex = ref(0)

onMounted(()=>{
  if(localStorage.getItem("progressMark") === null){
    localStorage.setItem("progressMark", JSON.stringify([]))
    subjectContents.value = JSON.parse(localStorage.getItem("progressMark"))
    showReset.value = true
  }
  else{
    subjectContents.value = JSON.parse(localStorage.getItem("progressMark"))
  }
})

onUpdated(()=>{
  if(subjectContents.value.length > 0){
    showReset.value = true
  }
  else{
    showReset.value = false
  }
})

function clear(){
  subjectData.value.subject = ''
  subjectData.value.progressMark = ''
  subjectData.value.markLeft = ''
  subjectData.value.passMark = ''
}

function removeAll(){
  if(confirm("remove all subjects?")){
    localStorage.setItem("progressMark", JSON.stringify([]))
    subjectContents.value = JSON.parse(localStorage.getItem("progressMark"))
  }
}

function validate(detail = {}){

  const valid = {valid:false}
  if(detail.subject && detail.progressMark
   && detail.markLeft && detail.passMark){
    error.value.error = false
    valid.valid = true
  }
  else{
    error.value.error = true
  }
  return valid.valid
}

function addSubject(event) {
  if (event) {
    const newSubjectContent = {
      subject:subjectData.value.subject, progressMark:subjectData.value.progressMark, 
      markLeft:subjectData.value.markLeft, passMark:subjectData.value.passMark
    }

    if(validate(newSubjectContent)){
      const storage = JSON.parse(localStorage.getItem("progressMark"))
      storage.push(newSubjectContent)
      localStorage.setItem("progressMark", JSON.stringify(storage))
      subjectContents.value = JSON.parse(localStorage.getItem("progressMark"))
    }
  } 
  clear() 
}

function neededToPass(passMark, progressMark){
  const needed = passMark - progressMark
  return needed
}

function deleteSubject(index){
  const removedSubject = subjectContents.value[index].subject
  if(confirm(`Confirm deletion of ${removedSubject}?`)){
    const newContent = subjectContents.value.filter((data) => data.subject !== subjectContents.value[index].subject);
    localStorage.setItem("progressMark", JSON.stringify(newContent))
    subjectContents.value = JSON.parse(localStorage.getItem("progressMark"))
  }
}

function updateSubject(index){
  subjectUpdateIndex.value = parseInt(index)
  showUpdateMenu.value = true
  updatedSUbjectData.value.subject = subjectContents.value[index].subject
  updatedSUbjectData.value.progressMark = subjectContents.value[index].progressMark
  updatedSUbjectData.value.markLeft = subjectContents.value[index].markLeft
  updatedSUbjectData.value.passMark = subjectContents.value[index].passMark
}

function editSubject(index){
  if(confirm("Confirm edit")){
    closeEdit()
    const newUpdatedSubjectContent = {
      subject:updatedSUbjectData.value.subject, progressMark:updatedSUbjectData.value.progressMark, 
      markLeft:updatedSUbjectData.value.markLeft, passMark:updatedSUbjectData.value.passMark
    }
    const storage = JSON.parse(localStorage.getItem("progressMark"))
    storage[index] = newUpdatedSubjectContent
    localStorage.setItem("progressMark", JSON.stringify(storage))
    subjectContents.value = JSON.parse(localStorage.getItem("progressMark"))
  }
}

function closeEdit(){
  showUpdateMenu.value = false
}

</script>

<template>
  <div class="heading">Progress Mark</div>
    <div class="subject_details">
      Add a new subject: <input v-model="subjectData.subject" class="subject" type="text">
      Progress Mark %: <input  v-model="subjectData.progressMark" class="percentages" type="number">
      Mark left %: <input  v-model="subjectData.markLeft" class="percentages" type="number">
      Pass Mark %: <input v-model="subjectData.passMark" class="percentages" type="number">
     <button class="add_button" @click="addSubject">Add</button>
    </div>
    <div v-show="showUpdateMenu" class="subject_details_update">
    <div class="update_heading">Edit</div>
     Subject: <input v-model="updatedSUbjectData.subject" class="subject" type="text">
      Progress Mark %: <input  v-model="updatedSUbjectData.progressMark" class="percentages" type="number">
      Mark left %: <input  v-model="updatedSUbjectData.markLeft" class="percentages" type="number">
      Pass Mark %: <input v-model="updatedSUbjectData.passMark" class="percentages" type="number">
     <button class="add_button" @click="editSubject(subjectUpdateIndex)">Edit</button>
     <button class="add_button" @click="closeEdit">Cancel</button>
    </div>
    <div v-show="error.error" class="error"> 
      {{ error.errorMessage }}
    </div>
    <div class="result_table">
      <table>
  <tr>
    <th>Subject</th>
    <th>Progress Mark(%)</th>
    <th>Needed to pass(%)</th>
    <th>Mark left(%)</th>
    <th>edit</th>
    <th>X</th>
  </tr>
  <tr v-for="(mark, index) in subjectContents" ref="outPut">
    <td>
      {{ mark.subject }}
    </td>
    <td>
      {{ mark.progressMark }}
    </td>
    <td>
      {{ neededToPass(mark.passMark ,mark.progressMark) }}
    </td>
    <td>
      {{ mark.markLeft }}
    </td>
    <td class="edit_button">
      <span @click="updateSubject(index)" class="material-symbols-outlined">update</span>
    </td>
    <td class="delete_button">
      <span @click="deleteSubject(index)" class="material-symbols-outlined">delete</span>
    </td>
  </tr>
</table>
    </div>
    <div v-show="showReset" class="reset"><button class="reset_button" @click="removeAll">Reset</button></div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Monoton&display=swap');

.heading{
  font-family: 'Monoton', cursive;
  color: rgb(18, 219, 18);
  text-align: center;
  font-size: 42px;
}

.update_heading{
  font-family: 'Monoton', cursive;
  color: rgb(18, 219, 18);
  text-align: center;
  font-size: 20px;
}
.subject_details{
  border: 5px solid rgb(18, 219, 18);
  color: rgb(18, 219, 18);
  font-size: 14pt;
  font-family:monospace;
  text-align: center;
  width: 92%;
  margin: auto;
}

.subject_details_update{
  border: 5px solid rgb(18, 219, 18);
  color: rgb(18, 219, 18);
  font-size: 14pt;
  font-family:monospace;
  text-align: center;
  width: 92%;
  margin: auto;
  margin-top: 1%;
}
.subject{
  border-top: none;
  border-left: none;
  border-right: none;
  border-bottom: 1px solid rgb(18, 219, 18);
  background-color: inherit;
  color:rgb(18, 219, 18);
  outline: none;
  font-size: medium;
  margin: 1%;
  width: 200px;
}

.percentages{
  border-top: none;
  border-left: none;
  border-right: none;
  border-bottom: 1px solid rgb(18, 219, 18);
  background-color: inherit;
  color:rgb(18, 219, 18);
  outline: none;
  font-size: medium;
  margin: 1%;
  width: 35px;
}
.add_button{
  color: rgb(204, 238, 204);
  background-color: green;
  cursor: pointer;
  width: 100px;
  height: fit-content;
}

.reset{
  margin-top: 2%;
  text-align: center;
}
.reset_button{
  color: rgb(204, 238, 204);
  background-color: rgb(216, 29, 29);
  cursor: pointer;
  width: 100px;
  height: fit-content;
}
.error{
  margin: 1%;
  color: rgb(243, 32, 32);
  font-size: 14pt;
  text-align: center;
}

table{
  color: rgb(18, 219, 18);
  font-family: monospace;
  font-size: 13pt;
  margin: 0 auto;
  margin-top: 25px;
  width: 70%;
}

.delete_button{
  text-align: center;
  cursor: pointer;
  color: red;
}

.edit_button{
  text-align: center;
  cursor: pointer;
  color: rgb(135, 207, 144);
}

table th, td{
  border: 2px solid rgb(18, 219, 18);
}

</style>
