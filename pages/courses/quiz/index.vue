<script setup lang="ts">
    definePageMeta({
        layout: 'course',
    })
    import { toast } from '@steveyuowo/vue-hot-toast'
    const delQuizName = ref()
    const delQuizId = ref()
    const userRole = useUserCourseState()
    const route = useRoute()
    const quizs = ref()
    const quizPending = ref(true)
    async function fetchQuiz(id: number) {
        await $fetch('/api/courses/view/', {
            query: {
                c_id: id,
                option: 'QUIZ',
            },
        })
            .then((res) => {
                quizs.value = res
                quizPending.value = false
            })
            .catch((err) => {
                toast.error(err?.data?.message)
                navigateTo('/mycourse', { replace: true })
            })
    }

    async function deleteQuiz(q_id: number) {
        const deleteQuizToast = toast.loading('กำลังลบแบบประเมินผล')

        await $fetch('/api/courses/quiz/', {
            method: 'DELETE',
            body: {
                q_id: delQuizId.value,
                c_id: route.query.id,
            },
        })
            .then((res) => {
                toast.update(deleteQuizToast, { type: 'success', message: res?.message })
                d_closeModal()
                fetchQuiz(route.query.id)
                quizPending.value = true
            })
            .catch((err) => {
                toast.update(deleteQuizToast, { type: 'error', message: err?.data?.message })
                navigateTo('/mycourse', { replace: true })
            })
    }

    if (route.query.id) {
        fetchQuiz(route.query.id)
    }

    function d_closeModal() {
        const { element } = HSOverlay.getInstance(quizDeleteModal.value, true)
        element.close()
    }

    function d_openModal() {
        const { element } = HSOverlay.getInstance(quizDeleteModal.value, true)
        element.open()
    }

    const quizDeleteModal = ref()

</script>
<template>
    <div
        ref="quizDeleteModal"
        id="quiz-delete-confirm"
        class="hs-overlay hs-overlay-open:opacity-100 hs-overlay-open:duration-500 hidden size-full fixed top-0 start-0 z-[80] opacity-0 overflow-x-hidden transition-all overflow-y-auto pointer-events-none">
        <div class="hs-overlay-open:opacity-100 hs-overlay-open:duration-500 opacity-0 transition-all sm:max-w-lg sm:w-full m-3 sm:mx-auto">
            <div class="flex flex-col bg-white border shadow-sm rounded-md pointer-events-auto">
                <div class="flex justify-between items-center py-3 px-4 border-b">
                    <h3 class="font-bold text-gray-800">ยืนยันการลบแบบทดสอบ</h3>
                    <button
                        type="button"
                        class="flex justify-center items-center size-7 text-sm font-semibold rounded-full border border-transparent text-gray-800 hover:bg-gray-100 disabled:opacity-50 disabled:pointer-events-none"
                        data-hs-overlay="#quiz-delete-confirm">
                        <span class="sr-only">Close</span>
                        <svg
                            class="flex-shrink-0 size-4"
                            xmlns="http://www.w3.org/2000/svg"
                            width="24"
                            height="24"
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round">
                            <path d="M18 6 6 18" />
                            <path d="m6 6 12 12" />
                        </svg>
                    </button>
                </div>
                <div class="flex flex-col p-6 gap-4">
                    <div>คุณกำลังจะลบ {{ delQuizName }}, คุณแน่ใจว่าต้องการลบใช่ไหม</div>
                </div>
                <div class="flex justify-end items-center gap-x-2 py-3 px-4">
                    <button
                        type="button"
                        class="py-2 px-3 inline-flex items-center gap-x-2 text-sm font-medium rounded-lg border border-gray-200 bg-white text-gray-800 shadow-sm hover:bg-gray-50 disabled:opacity-50 disabled:pointer-events-none"
                        data-hs-overlay="#quiz-delete-confirm">
                        ไม่ใช่
                    </button>
                    <button
                        type="button"
                        @click="deleteQuiz(delQuizId)"
                        class="transition-color duration-200 ease-in-out py-2 px-3 inline-flex items-center gap-x-2 text-sm font-semibold rounded-lg border border-transparent bg-rose-600 text-white hover:bg-rose-700 disabled:opacity-50 disabled:pointer-events-none">
                        ใช่
                    </button>
                </div>
            </div>
        </div>
    </div>

    <div class="flex flex-col gap-4 w-full">
        <div class="flex flex-row justify-between items-center gap-4">
            <div></div>
            <button
                v-if="userRole?.[route.query.id] !== 'STUDENT'"
                @click="navigateTo(`/courses/quiz/create?id=${route.query.id}`)"
                type="button"
                class="py-2 px-3 flex-shrink-0 transition-colors duration-150 ease-in-out inline-flex justify-center items-center gap-x-2 text-sm font-semibold rounded-lg border border-transparent bg-blue-600 text-white hover:bg-blue-700 disabled:opacity-50 disabled:pointer-events-none">
                <span class="material-icons-outlined">add</span>
                สร้างแบบทดสอบ
            </button>
        </div>
        <div v-if="(quizs?.length || 0) > 0" v-for="quiz in quizs" class="flex flex-row justify-between border border-1 rounded-md gap-2 w-full p-4">
            <div>
                <div class="flex flex-wrap items-center gap-4 w-full">
                    <div class="flex flex-col">
                        <span class="text-sm text-slate-400">{{ quiz.q_begin_date ? `เริ่มทำได้เมื่อ ${new Date(quiz.q_begin_date).toLocaleString()}` : '' }}</span>
                        <span class="text-sm text-slate-400">{{ quiz.q_due_date ? `กำหนดส่ง ${new Date(quiz.q_due_date).toLocaleString()}` : 'ไม่มีกำหนดส่ง' }}</span>
                    </div>
                </div>
                <div class="text-xl font-bold">{{ quiz.q_name }}</div>
            </div>
            <div class="flex items-center gap-2">
                <span v-if="userRole?.[route.query.id] !== 'STUDENT'" class="material-icons-outlined cursor-pointer select-none hover:text-blue-600">edit</span>
                <span
                    @click="
                        () => {
                            delQuizName = quiz.q_name
                            delQuizId = quiz.q_id
                            d_openModal()
                        }
                    "
                    v-if="userRole?.[route.query.id] !== 'STUDENT'"
                    class="material-icons-outlined cursor-pointer select-none hover:text-rose-600">
                    delete
                </span>
                <button
                    type="button"
                    class="py-2 px-3 flex-shrink-0 select-none transition-colors duration-150 ease-in-out inline-flex justify-center items-center gap-x-2 text-sm font-semibold rounded-lg border border-transparent bg-blue-600 text-white hover:bg-blue-700 disabled:opacity-50 disabled:pointer-events-none">
                    <span class="material-icons-outlined select-none">remove_red_eye</span>
                    ดูคะแนน
                </button>
            </div>
        </div>
        <div v-else-if="!quizPending && (quizs?.data?.length || 0) === 0" class="flex md:flex-row flex-col items-center border border-1 rounded-md gap-2 w-full p-4">
            <img class="w-48 p-4" src="/images/exam.svg" />
            <span class="text-3xl font-bold">ยังไม่มีแบบประเมินผลในคอร์สนี้</span>
        </div>
        <div v-else class="flex flex-row border border-1 rounded-md gap-2 w-full p-4">
            <div class="animate-spin inline-block size-6 border-[3px] border-current border-t-transparent text-blue-600 rounded-full dark:text-blue-500" role="status" aria-label="loading">
                <span class="sr-only">Loading...</span>
            </div>
            กำลังโหลดข้อมูล
        </div>
    </div>
</template>