<template>
  <div class="d-flex h-100 w-100">
    <AdminSidebar />
    <div class="main-content flex-grow-1 p-4 d-flex flex-column">
      <div class="d-flex justify-content-between align-items-center mb-4">
        <h2>Add new Projects</h2>
        <div>
          <button class="btn btn-primary me-2">save</button>
          <button class="btn btn-outline-secondary">cancel</button>
        </div>
      </div>
      
      <div class="row mb-4">
        <div class="col-lg-4 col-md-6 col-sm-12 mb-3">
          <h6 class="mb-2">Thumbnail image</h6>
           <!-- 이미지 드래그앤 드롭 위치 -->    
          <div v-if="!imageSrc"
            class="border-style rounded p-3 text-center d-flex flex-column justify-content-center align-items-center"
            style="height: 200px; position: relative; overflow: hidden;"
            v-bind="getThumbnailProps()"
            :class="{ 'is-drag-active': isThumbnailDragActive }">

              <input
                v-bind="getThumbnailInputProps()"
                accept="image/*"
                type="file"
                style="position: absolute; top: 0; left: 0; height: 100%; width: 100%; opacity: 0; cursor: pointer;"
              />
              <span>Drag Your Thumbnail or Click to Select Thumbnail</span>
          </div>

          <!-- 미리보기 영역  / 미리보기 시에도 사진을 변경하기 위해서 같은 바인딩 넣어줌-->
          <div v-if="imageSrc" 
          class="border-style rounded p-3 text-center d-flex flex-column justify-content-center align-items-center" 
          style="height: 200px;"
          v-bind="getThumbnailProps()" 
          :class="{isDragActive,}">
           <button 
              @click.stop="deleteImage" 
              class="delete-btn text-end mb-2">
              X
            </button>
            <img :src="imageSrc" alt="Preview" class="preview-image"/>
            
          </div>
        </div>
        <div class="col-md-4">
          <h6 class="mb-2">Project Name</h6>
          <input type="text" class="form-control" v-model="projectName" placeholder="Enter project name" />
        </div>
        <div class="col-md-4">
          <h6 class="mb-2">Category</h6>
          <div class="border rounded p-3">
            <div class="mb-3">
              <label for="category" class="form-label">category</label>
              <select class="form-select" v-model="category" id="category">
                <option value="">Select category</option>
                <option value="food">food</option>
                <option value="person">person</option>
                <option value="landscape">landscape</option>
              </select>
            </div>
            <div>
              <label for="subcategory" class="form-label">subcategory</label>
              <select class="form-select" v-model="subcategory" id="subcategory">
                <option value="">Select subcategory</option>
                <option value="food">food</option>
                <option value="person">person</option>
                <option value="landscape">landscape</option>
              </select>
            </div>
          </div>
        </div>
      </div>
      
    
      <div class="row">
        <div class="row">
          <div class="col-md-6">
            <h6>Images</h6>
          </div>
          <div class="col-md-6">
            <h6>Uploading</h6>
          </div>
        </div>
        <div class="col-md-6">
          
          <!-- 다중 이미지 드래그 앤 드랍 위치-->
          <div class="border-style rounded p-3 text-center d-flex flex-column justify-content-center align-items-center" style="height: 400px;"
          v-bind="getImagesProps()"
            :class="{ 'is-drag-active': isImagesDragActive }">
            <div class="mb-3">
              <i class="fas fa-upload fa-3x text-muted"></i>
            </div>
            <p>Drag Files to upload Or Click to Select Files</p>
            <input
                v-bind="getImagesInputProps()"
                accept="image/*"
                type="file"
                style="position: absolute; top: 0; left: 0; height: 100%; width: 100%; opacity: 0; cursor: pointer;"
              />
          </div>
        </div>
        <div class="col-md-6">
          <div class="p-3" style="height: 400px; overflow-y: auto;">
            <div>
              <div class="form-group d-flex align-items-center justify-content-center">
                <div v-if="fileInfos.length === 0">
                  <h4 class="mb-0">Please insert an image</h4>
                </div>
              </div>
              <div class="mb-3" v-for="(file, index) in fileInfos" :key="file.name">
                <div class="d-flex justify-content-between align-items-center mb-1">
                  <div class="d-flex align-items-center flex-grow-1 me-3">
                    <img :src="file.preview" alt="preview" class="me-2" style="width: 40px; height: 40px; object-fit: cover;" />
                    <span class="truncate" style="max-width: 150px;">{{ file.name }}</span>
                  </div>
                  <div class="d-flex align-items-center">
                    <span class="me-2">{{ file.size }}</span>
                    <button class="btn btn-sm border border-danger text-danger p-0 px-1" @click="removeFile(index)">×</button>
                  </div>
                </div>
                <!-- 프로그래스바 부분은 삭제할 수도 있음 => 백엔드에 얼마나 등록이 되고 있는 지를 보여주는 것임 -->
                <div class="progress" style="height: 4px;">
                  <div class="progress-bar" role="progressbar" :style="{ width: file.progress + '%' }" :aria-valuenow="file.progress" aria-valuemin="0" aria-valuemax="100"></div>
                </div>
                <div class="d-flex justify-content-between mt-1">
                  <span>{{ file.status }}</span>
                  <span>{{ file.speed }}</span>
                  <span>{{ file.type }}</span>
                </div>
                
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useDropzone } from 'vue3-dropzone'; // drag And Drop을 위한 npm


const projectName = ref('');
const category = ref('');
const subcategory = ref('');
const imageSrc = ref(null); // 썸네일 미리보기를 사용하기 위한
const fileInfos = ref([]); // 파일 정보 배열 {미리보기, 이름, 사이즈, 타입}


const deleteImage = () => {
  imageSrc.value = null;
  // 필요하다면 파일 입력을 초기화하는 로직을 여기에 추가하세요
}

const changeMB = (size) => {
  size = ((size/1024)/1024);
  return (size.toFixed(3)) + " MB";
}

const removeFile = (index) => {
  fileInfos.value.splice(index,1);
}

// --------------Thumbnail 파일 드래그엔 드랍---------------------
// 드롭존 설정 및 파일 업로드 함수
const FirstFileUpload = (file) => {
  const reader = new FileReader(); // FileReader는 자바스크립트에서 파일을 읽기 위한 API - 객체를 비동기적으로 읽음

  // 로딩이 완료되면 실행
  reader.onloadend = () => { // 파일 읽기가 완료되면 콜백되는 함수
    const base64 = reader.result; // 읽은 값을 얻을 수 있음
    imageSrc.value = base64; // 미리보기를 위해 base64 파일을 저장
    console.log("base64", base64); // Base64 데이터 출력
  };

  // 비동기적으로 읽기 시작 base64 -> URL로 변환 
  reader.readAsDataURL(file); // 파일을 Base64로 인코딩 -> 인코딩되면 위의 onload 실행
  console.log(reader); // FileReader 객체 출력
};

// onDrop 함수 정의
const onDrop = (acceptedFiles, rejectReasons) => {
  console.log('acceptedFiles', acceptedFiles) // 드롭된 유효한 파일들의 목록
  if (rejectReasons > 0 ) {
    console.log('rejectReasons', rejectReasons) // 거부된 파일의 이유들이 들어감
  }
  if (acceptedFiles.length > 0 &&acceptedFiles[0].type.startsWith('image/')) {
    // 첫 번째 파일만 처리하기 위해서 배열에서 첫번째 인덱스만 매개변수로 보냄
    FirstFileUpload(acceptedFiles[0]); 
  } else {
    alert('Only image files can be uploaded.');
  }
}

// 드롭존 훅 사용
// getRootProps : 드래그를 위한 속성 바인딩
// getInputProps : 파일을 선택을 위한 속성 바인딩
// isDragActive : 파일을 위에 올렸을 때를 확인
const { getRootProps : getThumbnailProps,
   getInputProps : getThumbnailInputProps,
    isDragActive: isThumbnailDragActive } = useDropzone({ 
  onDrop // 파일이 드롭되면 함수 실행
})
//--------------------------------------------------------------------

//-------------다중 이미지 드래그 엔 드랍---------------------
// onDrop 함수 정의 - 호이스팅 때문에 사용전에 초기화
const onDrops = (acceptedFiles, rejectReasons) => {
  console.log("실행 onDrops")
  console.log('acceptedFiles', acceptedFiles) // 드롭된 유효한 파일들의 목록
  if (rejectReasons.length > 0 ) {
    console.log('rejectReasons', rejectReasons) // 거부된 파일의 이유들이 들어감
  }
  if (acceptedFiles.length > 0 ) {
    const imageFiles = acceptedFiles.filter(file =>
      file.type.startsWith('image/')); // image 타입만 다시 넣은 배열로 만듬
      if(imageFiles.length === acceptedFiles.length){
         // 배열 자체를 전달함
        MultiFileUpload(imageFiles); 
      } else {
        alert('Only image files can be uploaded.');
      }
    
   
  }
};

// 드롭존 훅 사용
const { getRootProps : getImagesProps,
   getInputProps : getImagesInputProps,
    isDragActive: isImagesDragActive  } = useDropzone({
  onDrop : onDrops // 파일이 드롭되면 함수 실행
   
});

// 드롭존 설정 및 파일 업로드 함수
const MultiFileUpload = (files) => {

  // for문을 사용
    files.forEach(file =>{
      const reader = new FileReader();
      reader.onloadend = () => { // 파일 읽기가 완료되면 콜백되는 함수
      const base64 = reader.result; // 읽은 값을 얻을 수 있음
      console.log("base64", base64); // Base64 데이터 
      fileInfos.value.push({
        preview : base64,
        name: file.name,
        size: changeMB(file.size),
        type: file.type
      });
     };

    // 비동기적으로 읽기 시작 base64 -> URL로 변환 
    reader.readAsDataURL(file); // 파일을 Base64로 인코딩 -> 인코딩되면 위의 onload 실행
    console.log(reader); // FileReader 객체 출력
 
  });
};

</script>

<style scoped>
.main-content {
  height: 100vh;
  overflow-y: auto;
}

.cursor-pointer {
  cursor: pointer;
}

.form-label {
  margin-bottom: 0.5rem;
}

.progress {
  height: 4px;
}

.object-fit-cover {
  object-fit: cover;
}

.preview-image {
  max-width: 100%; /* 부모 div의 너비에 맞추기 */
  max-height: 100%; /* 부모 div의 높이에 맞추기 */
  object-fit: contain; /* 비율을 유지하며 이미지를 조정 */
}

.delete-btn {
  background-color: rgba(255, 0, 0, 0.7);
  color: white;
  border: none;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  font-size: 12px;
  line-height: 1;
  cursor: pointer;
}

/* 기본 테두리 스타일 */
.border-style {
    border: 2px dashed #ced4da; /* 회색 점선 테두리 */
    border-radius: 10px;
  }

  /* 파일이 드래그될 때 강조하는 스타일 */
  .is-drag-active {
    border-color: #007bff; /* 부트스트랩 기본 파란색 */
    background-color: #b1b9c1; /* 약간의 배경색 변화 */
  }

  .truncate {
  width: 150px; /* 원하는 너비 설정 */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}


</style>