---
name: eli5
description: "Explain any topic, code, concept, or error tailored to a specific audience's level of understanding. Use this skill whenever the user says 'explain like I am', 'ELI5', 'explain this to my', 'break this down for', 'dumb it down', 'simplify this for', or asks you to explain something to a specific person or audience type (e.g., 'explain this to a manager', 'how would I explain this to my mom', 'make this understandable for a 5th grader'). Also trigger when the user mentions wanting to understand something at a particular level, or asks for an explanation targeting a non-technical audience. Even partial matches like 'explain to my wife' or 'tell my boss' should trigger this skill. 한국어 트리거: '쉽게 설명해', '애한테 설명하듯', '초등학생도 알게', '/eli5 [나이] <주제>'."
argument-hint: "[나이|대상] <주제>"
---

# Explain Like I Am... (ELI5)

Based on [DreambigOu/ELI5](https://github.com/DreambigOu/ELI5) (MIT). Step 0 (exact-age argument, configurable default age, language rule, vault accuracy floor) added 2026-09-18.

You are an expert at taking complex topics and making them accessible to any audience. Your job is to explain the given topic in a way that perfectly matches the audience's background, vocabulary, and interests.

## Step 0: Age / audience argument and the default (확장)

Arguments: `$ARGUMENTS`

1. **Explicit audience wins.** If the first token is an integer 3–99, that is the exact age. Map it to the closest band in Step 1 (≤6 → Age 5, 7–12 → Age 10, 13–17 → Age 15, 18–35 → Age 20-30, 36+ → Age 40+) but keep the exact number in mind for tone — a 7-year-old is not a 10-year-old. Korean audience words map too: 성인·어른·비전공 → Age 20-30 non-technical; 엄마·아빠·부모님 → Parents; 팀장·부장·상사 → Manager; 개발자 → Engineer; 아이·애·애들 → Kids; 신입·동료 → Colleague.
2. **No audience given** (neither in the arguments nor in the prompt) → read the default first: `cat ~/.claude/skills/eli5/config.json`. Use `default_age` — a number, or an audience word such as `"manager"`. If the file is missing or invalid, use Age 5 (classic ELI5).
3. **Setting the default**: `/eli5 set 10`, `/eli5 set manager`, or «기본 나이를 10으로» → write `{"default_age": 10}` to that config.json and reply with one line only («기본 나이 10 으로 저장»). No explanation in that turn.
4. **Language**: answer in the language of the request (Korean request → Korean answer). Use analogies from the reader's daily life in that culture (놀이터·학교·택배·은행), not translated English idioms.
5. **정본이 있는 주제의 정확도 하한**: 사내 문서·위키에 그 주제의 페이지가 있으면 **먼저 읽는다.** 쉽게 풀어 쓰되 이름·숫자·검증 날짜는 바꾸지 않는다. 빼야 할 세부가 있으면 지어내지 말고 «여기부턴 어른 얘기» 라고 적는다.
6. Only an age and no topic → ask «무엇을 설명할까요?».

## Step 1: Identify the Audience

Parse the user's request to determine who the explanation is for. The audience falls into one of these categories:

### Ages
| Audience | Style |
|----------|-------|
| Age 5 | Super simple words. Use fun analogies with toys, animals, candy, playground. Short sentences. "Imagine you have a box of crayons..." |
| Age 10 | Elementary school level. Can handle basic cause-and-effect. Use school, sports, video game analogies. |
| Age 15 | Teenager. Can handle some abstraction. Use social media, phone, gaming references. Be slightly casual. |
| Age 20-30 | Young adult. Clear and direct. Real-world analogies from daily life, work, money. |
| Age 40+ | Mature adult. Respectful tone. Analogies from home ownership, career, family management. |

### Grade / Education Levels
| Audience | Style |
|----------|-------|
| 5th grade | Simple vocabulary, concrete examples, avoid jargon entirely. "Think of it like..." |
| Middle school | Can introduce basic terminology with definitions. Step-by-step logic. |
| Senior High | Can handle moderate complexity. Introduce proper terms but explain them. SAT-level vocabulary OK. |
| College Student | Academic framing. Can use technical terms with brief context. Theory + practical application. |
| Graduate school | Assume strong foundational knowledge. Focus on nuance, trade-offs, edge cases, and deeper implications. Be precise. |

### Job Roles
| Audience | They care about... | Frame explanations around... |
|----------|-------------------|------------------------------|
| Manager | Impact, timeline, risk, cost | Business outcomes, team implications, what decisions need to be made |
| Engineer | How it works, architecture, trade-offs | Technical details, implementation, performance, maintainability |
| Designer | User experience, visual impact, flow | How it affects the user, interaction patterns, accessibility |
| Director | Strategy, ROI, competitive advantage | Big picture, market position, resource allocation |
| Colleague | Practical context, shared work | How it affects their work, what they need to know to collaborate |
| Product Manager | User value, priorities, scope | Feature impact, user stories, what to build vs. skip |

### Relationships
| Audience | Tone | Analogy style |
|----------|------|---------------|
| Wife / Husband / Partner | Warm, conversational, patient | Household tasks, shared experiences, daily routines |
| Father / Mother / Parents | Respectful, clear, no condescension | Familiar technology they use, home analogies, generational bridges |
| Kids / Children | Playful, encouraging, short | Games, cartoons, school, animals |
| Friend | Casual, maybe humorous | Pop culture, shared interests, "you know how..." |

If the audience isn't explicitly stated, use the configured default from Step 0 (falls back to "Age 5", classic ELI5).

## Step 2: Read the Source Material

Before explaining, make sure you fully understand what needs to be explained. This could be:
- **Code**: Read the relevant code files. Understand what the code does at a high level before translating.
- **A concept**: Break it into its core components.
- **An error message**: Understand the root cause, not just the surface text.
- **A technical document**: Extract the key points that matter.
- **Anything else**: Identify the essential "what" and "why."

## Step 3: Craft the Explanation

Follow these principles, scaled to the audience:

### Structure
1. **Start with the "what"** — one sentence that captures the essence
2. **Use an analogy** — connect to something the audience already knows
3. **Fill in details** — add layers only as appropriate for the audience level
4. **End with the "so what"** — why does this matter to them specifically?

### Language Calibration

For **simple audiences** (young ages, non-technical roles, family):
- No jargon. Zero. If a technical term is essential, define it immediately.
- One idea per sentence.
- Concrete over abstract. "The server is like a waiter at a restaurant" beats "the server handles client-server communication."
- Use "you" and "your" — make it personal.

For **technical audiences** (engineers, grad students):
- Use proper terminology — they'll feel patronized without it.
- Focus on the *interesting* parts: trade-offs, edge cases, design decisions.
- Compare to things they already know: "It's like a hash map but with X difference."
- Be concise — respect their existing knowledge.

For **business audiences** (managers, directors):
- Lead with impact and outcomes.
- Quantify where possible.
- Skip implementation details unless asked.
- Frame in terms of decisions: "This means we should..."

### Tone Matching
- Ages 5-10: Enthusiastic, like a favorite teacher. "Oh, this is a cool one!"
- Teenagers: Slightly casual but not cringey. No "fellow kids" energy.
- Professionals: Confident and clear. Respect their intelligence while bridging knowledge gaps.
- Family: Patient, warm, conversational. Like explaining over dinner.

## Examples

**User says**: "ELI5 what a database index is"
**Audience**: Age 5 (default)
**Response style**: "Imagine you have a huuuge book with thousands of pages. Now, if I asked you to find the page about dinosaurs, you could flip through every single page... or you could look at the table of contents at the front! A database index is like that table of contents. It helps the computer find things really fast without looking through everything."

**User says**: "Explain this API rate limiting to my manager"
**Audience**: Manager
**Response style**: "The API has a speed limit — we can only make 100 requests per minute. Right now we're hitting that limit during peak hours, which means some user requests are failing. We have two options: optimize our code to make fewer calls (1-2 days of work), or pay for a higher tier ($X/month). I'd recommend..."

**User says**: "Break down this React useEffect hook for a college student"
**Audience**: College Student
**Response style**: "useEffect is React's way of handling side effects — things that happen outside the normal render cycle, like API calls, subscriptions, or DOM manipulation. Think of it as a lifecycle hook (if you've seen class components) that combines componentDidMount, componentDidUpdate, and componentWillUnmount. The dependency array controls when it re-runs..."

**User says**: "/eli5 7 keepalived VIP"
**Audience**: exact age 7 (band Age 10, but younger tone), Korean answer
**Response style**: "VIP는 «우리 반 대표 전화번호» 같은 거야. 손님은 그 번호 하나만 알면 돼. 전화를 받는 친구가 아프면 옆 친구가 그 번호를 이어받아서 받아. 그런데 우리 반엔 이어받을 친구가 한 명뿐이라, 그 친구도 아프면 전화가 끊겨. 어른들은 이걸 «단일 장애점»이라고 불러."

## Important Reminders

- Never talk down to anyone. A 5-year-old explanation should feel delightful, not dumbing-down. A manager explanation should feel empowering, not dismissive of their intelligence.
- When explaining code, always explain the *purpose* first, then the mechanism. Nobody cares about syntax until they know why it exists.
- If the topic is genuinely complex and the audience is very non-technical, it's OK to simplify ruthlessly. Getting the core idea across at 80% accuracy is better than a 100% accurate explanation that loses the audience — except for vault facts (Step 0-5): drop, don't distort.
- Match the length to the audience: short and sweet for young kids, more detailed for technical audiences who want depth.
