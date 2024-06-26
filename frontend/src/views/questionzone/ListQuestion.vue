<template>
    <div class="container">
        <input class="search-bar" type="text" v-model="searchTerm" placeholder="검색..." />
        <div class="list-container">
            <div class="list-item" v-for="question in filteredQuestions" :key="question.id">
                <p>User ID: {{ question.user_id }}</p>
                <p>Question: {{ question.question_text }}</p>
                <!-- 문제 풀기 버튼 -->
                <button class="action-button" @click="goToSolvingQuestion(question.id)">문제 풀기</button>
                <!-- 토론장 이동 버튼 -->
                <button class="action-button" @click="goToDiscussQuestion(question.id)">토론장으로</button>
                <!-- 삭제 버튼 -->
                <button class="action-button" @click="deleteQuestion(question.id)">문제 삭제</button>
            </div>
        </div>
        <button class="add-button" @click="addQuestion">문제 추가</button>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    name: 'QuestionList',
    data(){
        return {
            questions: [],
            searchTerm: '',
            qualification_type: ''
        };
    },
    computed: {
      filteredQuestions() {
          return this.questions.filter(question =>
              question.question_text && // question_text가 null 또는 undefined가 아닌 경우에만 진행합니다.
              question.question_text.toLowerCase().includes(this.searchTerm.toLowerCase())
          );
      }
    },
    methods: {
        addQuestion() {
            this.$router.push({ name: 'addquestion', query: { category: this.qualification_type } });
        },
        goToSolvingQuestion(questionId) {
            this.$router.push({ name: 'solvingquestion', params: { questionId } });
        },
        goToDiscussQuestion(questionId) {
            this.$router.push({ name: 'discussquestion', params: { questionId } });
        },
        loadQuestions() {
            axios.get(`${process.env.VUE_APP_BACKEND_API}/question?qualification_type=${this.qualification_type}`, {
                headers: {
                    'Content-Type': 'application/json',
                }
            }).then(response => {
                this.questions = response.data.data.map(item => ({
                    id: item._id,
                    user_id: item.user_id,
                    question_text: item.question_text
                }));
            }).catch(error => {
                console.error('Error loading questions:', error);
            });
        },
        deleteQuestion(questionId){
            const userId = localStorage.getItem('user_id'); // 로컬 스토리지에서 사용자 ID 가져오기
            const index = this.questions.findIndex(question => question.id === questionId);
            if (index !== -1){
                axios.delete(`${process.env.VUE_APP_BACKEND_API}/question/delete?_id=${questionId}&user_id=${userId}`)
                    .then(response => {
                        if (response.status === 200){
                            this.questions.splice(index, 1);
                            alert('질문이 정상적으로 삭제되었습니다');
                        }
                    })
                    .catch(error => {
                      if (error.response.status === 403) {
                        alert('본인이 등록한 문제만 삭제할 수 있습니다');
                      } else {
                        console.error('Error deleting the question:', error);
                        alert('Failed to delete the question');}
                    });
            }
        },
    },
    created(){
        this.qualification_type = decodeURIComponent(this.$route.query.category || '');
        this.loadQuestions(); // 데이터 로딩을 위해 메서드 호출
    }
};
</script>

<style scoped>
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
}

.search-bar {
    margin-bottom: 20px;
    width: 100%;
    /* 가로 전체 길이 사용 */
    padding: 10px;
    border: 2px solid #ccc;
    /* 그레이 테두리 */
    border-radius: 5px;
    /* 약간의 둥근 모서리 */
}

.list-container {
    width: 100%;
    /* 가로 전체 길이 사용 */
    background-color: #f9f9f9;
    /* 밝은 그레이 배경 */
    border: 1px solid #ddd;
    /* 섬세한 테두리 */
    border-radius: 5px;
    max-height: 300px;
    /* 최대 높이 설정 */
    overflow-y: auto;
    /* 내용이 넘칠 경우 스크롤 */
}

.list-item {
    padding: 15px;
    border-bottom: 1px solid #eee;
    /* 항목 간 구분선 */
}

.list-item:last-child {
    border-bottom: none;
    /* 마지막 항목은 구분선 제거 */
}

.add-button {
    margin-top: 20px;
    padding: 10px 15px;
    background-color: #333;
    /* 어두운 배경 */
    color: white;
    /* 텍스트 색상 */
    border: none;
    /* 테두리 제거 */
    border-radius: 5px;
    /* 둥근 모서리 */
    cursor: pointer;
    /* 클릭 가능한 핸드 커서 */
    font-weight: bold;
    /* 글자 진하게 */
}

.add-button:hover {
    background-color: #555;
    /* 마우스 호버시 배경색 변경 */
}

.action-button {
    margin-top: 10px;
    padding: 5px 10px;
    background-color: #333;
    /* Green */
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8rem;
}

.action-button:hover {
    background-color: #333;
}
</style>