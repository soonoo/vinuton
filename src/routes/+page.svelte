<script lang="ts">
	import { goto } from '$app/navigation';

	const services = [
		{
			id: 'email',
			title: '이메일 교정 도우미',
			description: '교수님께 보낼 이메일을 더 정중하게 다듬어 드립니다',
			icon: '✉️',
			path: '/email',
			color: 'blue'
		},
		{
			id: 'resume',
			title: '자기소개서/이력서 첨삭',
			description: 'AI가 여러분의 자기소개서를 분석하고 개선점을 제안합니다',
			icon: '📄',
			path: '/resume',
			color: 'indigo'
		},
		{
			id: 'job-interview',
			title: '면접 예상 질문 생성기',
			description: '전공과 수강 과목을 기반으로 맞춤형 면접 질문을 생성합니다',
			icon: '💼',
			path: '/job-interview',
			color: 'purple'
		},
		{
			id: 'meeting-minutes',
			title: '회의록 자동 생성기',
			description: '회의 내용을 녹음하면 자동으로 요약본을 만들어 드립니다',
			icon: '🎙️',
			path: '/meeting-minutes',
			color: 'red'
		}
	];
</script>

<div class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-100 py-12 px-4 sm:px-6 lg:px-8">
	<div class="max-w-4xl mx-auto">
		<!-- 헤더 -->
		<div class="text-center mb-12">
			<h1 class="text-4xl font-bold text-gray-900 mb-4">학생 도우미 서비스</h1>
			<p class="text-xl text-gray-600">대학생활에 필요한 글쓰기를 도와드립니다</p>
		</div>

		<!-- 서비스 카드 -->
		<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
			{#each services as service}
				<button
					onclick={() => goto(service.path)}
					class="group relative bg-white rounded-xl shadow-md hover:shadow-xl transition-all duration-300 p-8 text-left overflow-hidden"
				>
					<!-- 배경 그라데이션 -->
					<div class="absolute inset-0 bg-gradient-to-br opacity-0 group-hover:opacity-5 transition-opacity duration-300
						{service.color === 'blue' ? 'from-blue-400 to-blue-600' : 
						 service.color === 'indigo' ? 'from-indigo-400 to-indigo-600' : 
						 service.color === 'purple' ? 'from-purple-400 to-purple-600' : 
						 'from-red-400 to-red-600'}">
					</div>

					<!-- 아이콘 -->
					<div class="text-5xl mb-4">{service.icon}</div>

					<!-- 제목과 설명 -->
					<h2 class="text-2xl font-bold text-gray-900 mb-2 group-hover:text-{service.color}-600 transition-colors">
						{service.title}
					</h2>
					<p class="text-gray-600 leading-relaxed">
						{service.description}
					</p>

					<!-- 화살표 아이콘 -->
					<div class="absolute bottom-4 right-4 opacity-0 group-hover:opacity-100 transform translate-x-2 group-hover:translate-x-0 transition-all duration-300">
						<svg class="w-6 h-6 text-{service.color}-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
						</svg>
					</div>
				</button>
			{/each}
		</div>

		<!-- 추가 정보 -->
		<div class="mt-12 text-center">
			<div class="inline-flex items-center justify-center space-x-2 text-sm text-gray-500">
				<svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
				</svg>
				<span>모든 데이터는 안전하게 처리되며 저장되지 않습니다</span>
			</div>
		</div>
	</div>
</div>

<style>
	/* Tailwind 클래스가 동적으로 생성되는 경우를 위한 추가 스타일 */
	.group:hover .group-hover\:text-blue-600 {
		color: rgb(37 99 235);
	}
	
	.group:hover .group-hover\:text-indigo-600 {
		color: rgb(79 70 229);
	}

	.text-blue-600 {
		color: rgb(37 99 235);
	}

	.text-indigo-600 {
		color: rgb(79 70 229);
	}

	.from-blue-400 {
		--tw-gradient-from: rgb(96 165 250);
	}

	.to-blue-600 {
		--tw-gradient-to: rgb(37 99 235);
	}

	.from-indigo-400 {
		--tw-gradient-from: rgb(129 140 248);
	}

	.to-indigo-600 {
		--tw-gradient-to: rgb(79 70 229);
	}

	.group:hover .group-hover\:text-purple-600 {
		color: rgb(147 51 234);
	}

	.text-purple-600 {
		color: rgb(147 51 234);
	}

	.from-purple-400 {
		--tw-gradient-from: rgb(192 132 252);
	}

	.to-purple-600 {
		--tw-gradient-to: rgb(147 51 234);
	}

	.group:hover .group-hover\:text-red-600 {
		color: rgb(220 38 38);
	}

	.text-red-600 {
		color: rgb(220 38 38);
	}

	.from-red-400 {
		--tw-gradient-from: rgb(248 113 113);
	}

	.to-red-600 {
		--tw-gradient-to: rgb(220 38 38);
	}
</style>
