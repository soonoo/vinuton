<script lang="ts">
	import { onMount } from 'svelte';

	// 더미 사용자 데이터 (실제로는 로그인된 사용자 정보에서 가져올 예정)
	const userInfo = {
		name: '김학생',
		major: '컴퓨터공학과',
		year: '4학년',
		courses: [
			'자료구조',
			'알고리즘',
			'운영체제',
			'데이터베이스',
			'소프트웨어공학',
			'컴퓨터네트워크',
			'인공지능개론',
			'웹프로그래밍',
			'모바일프로그래밍',
			'캡스톤디자인'
		]
	};

	let questions = $state<string[]>([]);
	let showQuestions = $state(false);
	let selectedCompanyType = $state('tech');
	let selectedPosition = $state('backend');

	const companyTypes = [
		{ value: 'tech', label: 'IT/테크 기업' },
		{ value: 'finance', label: '금융권' },
		{ value: 'startup', label: '스타트업' },
		{ value: 'manufacturing', label: '제조업' },
		{ value: 'public', label: '공기업' }
	];

	const positions = [
		{ value: 'backend', label: '백엔드 개발자' },
		{ value: 'frontend', label: '프론트엔드 개발자' },
		{ value: 'fullstack', label: '풀스택 개발자' },
		{ value: 'data', label: '데이터 엔지니어' },
		{ value: 'ai', label: 'AI/ML 엔지니어' },
		{ value: 'devops', label: 'DevOps 엔지니어' }
	];

	function generateQuestions() {
		// 더미 질문 생성 로직 (나중에 API로 대체)
		const baseQuestions = [
			`${userInfo.major}를 전공하게 된 계기와 가장 흥미로웠던 수업은 무엇인가요?`,
			`${userInfo.courses.includes('알고리즘') ? '알고리즘 수업에서 배운 내용 중 실무에 어떻게 적용할 수 있을까요?' : '프로그래밍 문제를 해결하는 당신만의 접근 방법은 무엇인가요?'}`,
			'팀 프로젝트에서 갈등이 생겼을 때 어떻게 해결했나요?',
			`${selectedPosition === 'backend' ? 'RESTful API 설계 시 고려해야 할 사항은 무엇인가요?' : '사용자 경험(UX)을 개선하기 위한 방법은 무엇인가요?'}`,
			'자신의 강점과 약점을 각각 한 가지씩 말씀해주세요.',
			`${userInfo.courses.includes('데이터베이스') ? '데이터베이스 정규화의 장단점에 대해 설명해주세요.' : '데이터를 효율적으로 관리하는 방법에 대해 아는 대로 설명해주세요.'}`,
			'최근 관심있게 본 기술 트렌드는 무엇이고, 왜 관심을 갖게 되었나요?',
			`${selectedCompanyType === 'startup' ? '스타트업에서 일하고 싶은 이유는 무엇인가요?' : '대기업에서 일하고 싶은 이유는 무엇인가요?'}`,
			'프로젝트를 진행하면서 가장 어려웠던 기술적 문제와 해결 방법을 설명해주세요.',
			'5년 후 어떤 개발자가 되어있고 싶나요?'
		];

		// 전공과 수강 과목에 따른 추가 질문
		if (userInfo.courses.includes('인공지능개론')) {
			baseQuestions.push('AI 기술이 사회에 미치는 영향에 대해 어떻게 생각하시나요?');
		}

		if (userInfo.courses.includes('캡스톤디자인')) {
			baseQuestions.push('캡스톤 프로젝트에서 담당한 역할과 기여한 부분을 설명해주세요.');
		}

		questions = baseQuestions.slice(0, 10);
		showQuestions = true;
	}

	function copyQuestion(question: string) {
		navigator.clipboard.writeText(question);
	}
</script>

<div class="min-h-screen bg-gray-50 py-8 px-4 sm:px-6 lg:px-8">
	<div class="max-w-4xl mx-auto">
		<div class="text-center mb-8">
			<h1 class="text-3xl font-bold text-gray-900">면접 예상 질문 생성기</h1>
			<p class="mt-2 text-gray-600">전공과 수강 과목을 기반으로 맞춤형 면접 질문을 생성합니다</p>
		</div>

		<!-- 사용자 정보 표시 -->
		<div class="bg-white rounded-lg shadow-md p-6 mb-6">
			<h2 class="text-lg font-semibold text-gray-900 mb-4">내 정보</h2>
			<div class="grid md:grid-cols-2 gap-4">
				<div>
					<p class="text-sm text-gray-600">학과</p>
					<p class="font-medium text-gray-900">{userInfo.major} {userInfo.year}</p>
				</div>
				<div>
					<p class="text-sm text-gray-600">주요 수강 과목</p>
					<div class="flex flex-wrap gap-2 mt-1">
						{#each userInfo.courses.slice(0, 5) as course}
							<span class="px-2 py-1 bg-purple-100 text-purple-700 text-xs rounded-full">
								{course}
							</span>
						{/each}
						{#if userInfo.courses.length > 5}
							<span class="px-2 py-1 bg-gray-100 text-gray-600 text-xs rounded-full">
								+{userInfo.courses.length - 5}개
							</span>
						{/if}
					</div>
				</div>
			</div>
		</div>

		<!-- 옵션 선택 -->
		<div class="bg-white rounded-lg shadow-md p-6 space-y-6">
			<div class="grid md:grid-cols-2 gap-6">
				<div>
					<label for="company-type" class="block text-sm font-medium text-gray-700 mb-2">
						지원 기업 유형
					</label>
					<select
						id="company-type"
						bind:value={selectedCompanyType}
						class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent"
					>
						{#each companyTypes as type}
							<option value={type.value}>{type.label}</option>
						{/each}
					</select>
				</div>

				<div>
					<label for="position" class="block text-sm font-medium text-gray-700 mb-2">
						지원 직무
					</label>
					<select
						id="position"
						bind:value={selectedPosition}
						class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent"
					>
						{#each positions as pos}
							<option value={pos.value}>{pos.label}</option>
						{/each}
					</select>
				</div>
			</div>

			<!-- 질문 생성 버튼 -->
			<div class="text-center">
				<button
					onclick={generateQuestions}
					class="px-8 py-3 bg-purple-600 text-white font-medium rounded-lg hover:bg-purple-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-500 transition duration-200"
				>
					예상 질문 생성하기
				</button>
			</div>
		</div>

		<!-- 생성된 질문 목록 -->
		{#if showQuestions}
			<div class="mt-6 bg-white rounded-lg shadow-md p-6 animate-fade-in">
				<h3 class="text-lg font-semibold text-gray-900 mb-4">예상 면접 질문</h3>
				<div class="space-y-3">
					{#each questions as question, index}
						<div class="group relative bg-purple-50 rounded-lg p-4 hover:bg-purple-100 transition-colors">
							<div class="flex items-start space-x-3">
								<span class="flex-shrink-0 w-7 h-7 bg-purple-600 text-white text-sm font-medium rounded-full flex items-center justify-center">
									{index + 1}
								</span>
								<p class="flex-1 text-gray-800 leading-relaxed pr-8">
									{question}
								</p>
								<button
									onclick={() => copyQuestion(question)}
									class="absolute right-3 top-3 opacity-0 group-hover:opacity-100 transition-opacity p-2 hover:bg-purple-200 rounded"
									title="질문 복사"
								>
									📋
								</button>
							</div>
						</div>
					{/each}
				</div>

				<!-- 팁 섹션 -->
				<div class="mt-6 bg-amber-50 rounded-lg p-4">
					<h4 class="font-semibold text-amber-900 mb-2">💡 면접 준비 팁</h4>
					<ul class="space-y-1 text-sm text-amber-800">
						<li>• STAR 기법(상황-과제-행동-결과)을 활용해 답변을 구성하세요</li>
						<li>• 각 질문에 대해 1-2분 분량의 답변을 준비하세요</li>
						<li>• 구체적인 경험과 사례를 포함해 답변하세요</li>
						<li>• 거울을 보며 연습하거나 녹음해서 들어보세요</li>
					</ul>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	@keyframes fade-in {
		from {
			opacity: 0;
			transform: translateY(10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.animate-fade-in {
		animation: fade-in 0.3s ease-out;
	}
</style>