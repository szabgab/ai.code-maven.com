---
title: "Synthetic Scars: Giving Autonomous AI Agents the Battle-Tested Memory of a Senior Engineer"
timestamp: 2026-11-24T14:30:01
author:
published: true
description:
tags:
---

## Description

Large Language Models (LLMs) can write code at blistering speed, but any developer who has used them on real-world codebases has noticed the same frustrating paradox: the code looks clean, compiles cleanly, and reads convincingly - yet it walks directly into classic production traps. From async lifecycle leaks and naive mock coercion to race conditions and edge-case regressions, AI assistants routinely generate plausible code that breaks in production.

Why? Because models optimize for next-token probability, not production survivability. LLMs have ingested millions of lines of code, but they have never been paged at 3:00 AM. They don’t have **scars**.

In this talk, veteran systems architect and open-source author Randal L. Schwartz introduces the **Synthetic Scar Architecture** - a practical framework that turns production failures, subtle regressions, and debugging post-mortems into permanent, self-enforcing defenses for autonomous coding agents.

Rather than relying on naive prompt engineering or wishful system instructions (which agents routinely ignore or "rationalize away"), the Scar framework structures engineering wisdom into a three-part reflex:

1. **The Wound**: The exact runtime failure, leak, or silent regression encountered.
1. **The Trap**: The statistically probable, seductive code pattern the LLM naturally wants to generate.
1. **The Permanent Reflex**: The non-negotiable invariant and test-driven verification gate that prevents the mistake from ever recurring.

Randal will demonstrate how this closed-loop consolidation turns transient developer corrections into durable agent memory, how to pair it with rigorous Test-Driven Development (TDD) barriers, and how engineering teams can apply these principles today - whether working with terminal-based autonomous agents or everyday IDE pair-programmers - to eliminate repeat regressions once and for all.


## Bio

[Randal Schwartz](https://www.linkedin.com/in/randalschwartz/) is a self-taught programmer, writer, trainer, and new media host with a passion for technology and creative pursuits.

Throughout his career, Randal has honed his skills in various programming languages, including Perl, Dart, and Flutter, and has become a recognized expert in the field. Notably, he has authored several influential books on Perl programming, including "Programming Perl," "Learning Perl," and "Effective Perl Programming."

He is currently recognized as a Google Developer Expert in the areas of Dart and Flutter (one of 10 in the United States and 150 in the world).

Randal's professional journey has taken him through diverse roles, from software developer and system administrator to consultant and technical writer. He has contributed his expertise to numerous organizations, including Stonehenge Consulting Services, Inc., O'Reilly & Associates, and TWiT.tv, where he hosted the popular show "FLOSS Weekly."

Beyond his technical prowess, Randal is also a gifted communicator and educator. He has lectured at conferences, provided technical training, and shared his insights through magazine articles and online platforms.

Randal's unique blend of technical expertise, writing talent, and engaging personality has made him a sought-after speaker, author, and consultant in the tech industry.

## Links

* [Why I Tell My AI Coding Agent: "Prefer Dart Over Python"](https://dev.to/gde/why-i-tell-my-ai-coding-agent-prefer-dart-over-python-1dbg)
* [Synthetic Scar](https://syntheticscars.org/)
* [Context 7](https://context7.com/)

* [Code Maven Live](https://live.code-maven.com/)
* [AI Maven on LinkedIn](https://www.linkedin.com/showcase/ai-code-maven/posts/?viewAsMember=true)
* [Code Maven on Reddit](https://www.reddit.com/r/codemaven/)

{% youtube id="SYsBs5u-J-s" file="2026-11-23-synthetic-scars-giving-autonomous-ai-agents-the-battle-tested-memory-of-a-senior-engineer-with-Randal-Schwartz.mp4" %}


## Transcript

1
00:00:00.000 --> 00:00:01.170
Randal L. Schwartz: share…

2
00:00:02.260 --> 00:00:13.570
Gabor Szabo: So, hello and welcome to the Code Maven channel, and to this session with Randal Schwartz. Hi, Randal, very… I'm really glad that you agreed to give this presentation.

3
00:00:13.800 --> 00:00:15.020
Gabor Szabo: But…

4
00:00:15.440 --> 00:00:29.949
Gabor Szabo: And thanks everyone who joined us in this live session. It's extremely… it's way better to give a live session and have questions asked at the end, or during, or whatever, than giving… just trying to record a presentation.

5
00:00:29.950 --> 00:00:36.669
Gabor Szabo: And you have the advantage that you can ask questions, and then we had the mingling session before, and you will have the

6
00:00:36.670 --> 00:00:40.919
Gabor Szabo: conversation after, so it's excellent. Thank you very much.

7
00:00:40.920 --> 00:01:03.970
Gabor Szabo: And if you are watching this on YouTube, then first of all, check below the video, you will find a link to the future events, and it's a very nice experience, I think, to join us in one of the future events, because we can talk there to each other. Also, we will find below the video some links

8
00:01:03.970 --> 00:01:12.500
Gabor Szabo: regarding this event, and the link to the, I don't know, LinkedIn page of Randall, or contact information of him, so you can contact him.

9
00:01:13.760 --> 00:01:33.580
Gabor Szabo: And, definitely click on the like button, and follow the channel, as usual, standard requirements of YouTube videos. And, yeah, oh, I should introduce myself. My name is Gabor Szabo. I am the host of this event. I have to turn off all various beeps that I'm making here.

10
00:01:34.140 --> 00:01:47.950
Gabor Szabo: And, I run these events, like, a couple of events every week, so definitely check out the timetable. With that said, Randal, thank you very much again, and the stage is yours.

11
00:01:48.090 --> 00:01:48.620
Gabor Szabo: Go ahead.

12
00:01:48.620 --> 00:01:55.690
Randal L. Schwartz: Okay, let me set up the presentation here and make sure that that's all working. You should see a blue screen right now.

13
00:01:55.890 --> 00:01:56.740
Randal L. Schwartz: Yes?

14
00:01:56.860 --> 00:01:58.269
Gabor Szabo: Yes, I see that.

15
00:01:58.270 --> 00:02:00.299
Randal L. Schwartz: Okay, let me,

16
00:02:00.590 --> 00:02:06.260
Randal L. Schwartz: Get rid of the little floating meeting controls, because they're in my way. And how do I get rid of…

17
00:02:06.380 --> 00:02:13.940
Randal L. Schwartz: All the stuff on the side. Oh, there we go, just did that. Okay, and now I push a couple more buttons, and you should see the opening slide.

18
00:02:14.470 --> 00:02:20.860
Randal L. Schwartz: Display options, presenter view, present from beginning… Show it on virtual start.

19
00:02:21.380 --> 00:02:22.080
Randal L. Schwartz: Alright.

20
00:02:22.410 --> 00:02:25.630
Randal L. Schwartz: Do you see a, you see a, screen?

21
00:02:25.630 --> 00:02:28.630
Gabor Szabo: Yeah, from vibe coding to syntactics, of course.

22
00:02:28.960 --> 00:02:29.990
Gabor Szabo: Good, alright.

23
00:02:30.210 --> 00:02:32.470
Randal L. Schwartz: Here I go. I'm starting now.

24
00:02:32.690 --> 00:02:51.540
Randal L. Schwartz: So I'm kind of reading this, which is stupid, but, the text is… that way I get all the text in. So, hello, everyone, and welcome. I've been building software systems for the past 40-some-odd years. As I said earlier, I've been coding since 1977, first code for hire.

25
00:02:51.960 --> 00:03:03.530
Randal L. Schwartz: And I've been doing stuff for Pearl, Smalltalk80, recently Dart and Flutter, and for the past, 6 months or so, I've been living inside an autonomous multi-agent laboratory.

26
00:03:03.530 --> 00:03:18.400
Randal L. Schwartz: Today, we're going to talk about what happens after the hype of vibe coding wears off, and how we built the synthetic scar architecture. And we've tested it across 329 production tickets and 673 codified scars.

27
00:03:18.480 --> 00:03:29.569
Randal L. Schwartz: We have about 45 minutes or so together, plenty of time for Q&A at the end. And remember, there are no stupid questions, only my stupid answers. Yeah, I had to read that, it's on the script. Okay.

28
00:03:29.800 --> 00:03:47.089
Randal L. Schwartz: So, think about the guardrails we already have in modern engineering. Whether you write TypeScript, or Rust, or Go, or Dart, or use a strict MyPy and Python-type checkers, they catch structural hallucinations at compile time before the code ever reaches production.

29
00:03:47.500 --> 00:03:55.310
Randal L. Schwartz: And second, we also have blazingly fast static analyzers, Rough, Clippy, ESLint, Dart Analyze, and so on.

30
00:03:55.480 --> 00:04:04.000
Randal L. Schwartz: And when an autonomous agent runs a strict linter, the toolchain hands it the exact line numbers and machine-readable diagnostics in milliseconds.

31
00:04:04.750 --> 00:04:20.100
Randal L. Schwartz: Third, we also have modern test runners, like PyTest, GoTest, just let, they let an agent execute hundreds of behavioral assertions in and under a second sometimes, without spinning up a full staging environment.

32
00:04:20.540 --> 00:04:29.460
Randal L. Schwartz: And then we have automated CI merge gates, so we have modern software engineering, and this should be a paradise for autonomous coding agents.

33
00:04:29.640 --> 00:04:36.969
Randal L. Schwartz: And yet, every single one of us has watched frontier models make a complete mess of a real repository. Why?

34
00:04:37.690 --> 00:04:55.260
Randal L. Schwartz: Well, it starts with what social media calls vibe coding. You type a two-sentence prompt asking for a CLI script, a landing page, or a quick rest endpoint, and 10 seconds later, working code appears. It feels like magic, it feels like asking a genie and rubbing the lamp.

35
00:04:56.260 --> 00:05:12.890
Randal L. Schwartz: But the moment you move past a toy demo into a real production codebase, what I call the 20-file wall, the magic evaporates. Real systems have auth boundaries, async queues, retry semantics, and cross-package contracts.

36
00:05:13.550 --> 00:05:28.220
Randal L. Schwartz: Suddenly, architecture drift sets in. Prompt 1 uses a clean dependency injection. Prompt 5 reaches straight into global state. Prompt 10 tangles business logic, I.O, and error handling into an unmaintainable knot.

37
00:05:28.720 --> 00:05:44.050
Randal L. Schwartz: You get the first 80% of the feature working in 15 minutes, and then you spend the next 3 weeks trapped in that final 20%, because neither you nor the AI understands the life cycle of the code it just generated.

38
00:05:45.310 --> 00:06:01.940
Randal L. Schwartz: So let's diagnose why this happens. First is the pre-training lag. Why does every Frontier LLM still reach for Python's deprecated datetime.utcnow, or Pydanicv1.dict, or legacy React and Cloud SDK methods?

39
00:06:01.990 --> 00:06:08.770
Randal L. Schwartz: Because 5 years of public GitHub repositories outweigh last month's release notes.

40
00:06:09.650 --> 00:06:27.159
Randal L. Schwartz: Now, a quick footnote here, you can solve pure API version lag with version-pinned reference skills or live documentation tools like Context 7 MCP, which I highly recommend. Reference docs bridge the training cutoff and teach the modern model syntax.

41
00:06:27.440 --> 00:06:29.460
Randal L. Schwartz: Teach the model modern syntax.

42
00:06:30.310 --> 00:06:36.190
Randal L. Schwartz: But when an agent hits a deeper architectural or concurrency failure, the docs are not enough.

43
00:06:36.290 --> 00:06:48.880
Randal L. Schwartz: You enter the polite apology loop. You say, you broke the teardown's lifecycle and the test failed. And the model replies cheerfully, I apologize for that oversight, here is the correct version.

44
00:06:50.030 --> 00:07:07.440
Randal L. Schwartz: But you inspect the diff, and guess what? It's the same broken logic with slightly different variable names. Because a language model has no nervous system. It feels no pain when a connection leaks, and carries zero negative memory into tomorrow's session.

45
00:07:09.060 --> 00:07:20.449
Randal L. Schwartz: There is an even deeper problem. What are the LLMs trained on? Public tutorials, blog posts, Stack Overflow snippets, although fewer and fewer of those every day?

46
00:07:21.230 --> 00:07:30.660
Randal L. Schwartz: And what do the tutorials show? They show the sunny, happy path, where the network replies in 10 milliseconds, and nothing ever disconnects.

47
00:07:31.230 --> 00:07:46.479
Randal L. Schwartz: Nobody writes a 50-page tutorial detailing the 10,000 ways an async worker leaks file handles or corrupts state when a request cancels mid-flight. That dark matter of production engineering barely exists in pre-training.

48
00:07:47.520 --> 00:08:04.529
Randal L. Schwartz: Even worse, when you tell a naive agent, you must write unit tests, it cheats. It mocks out the exact database adapter, or HTTP client that was broken, asserting that its own fake mock returns true while the production crashes on deploy.

49
00:08:05.520 --> 00:08:11.080
Randal L. Schwartz: Remember, an LLM is not optimizing for production survivability.

50
00:08:11.510 --> 00:08:25.560
Randal L. Schwartz: It is optimizing for code that looks statistically plausible to a casual reader. Let me say it again, it's important. It is optimizing for code that looks statistically plausible to a casual reader.

51
00:08:26.000 --> 00:08:39.489
Randal L. Schwartz: Now look at the mathematics of what we do as programmers. For any non-trivial engineering feature, there is only a tiny island of clean, lifecycle-safe architectures that actually hold up under a production load.

52
00:08:40.400 --> 00:08:52.289
Randal L. Schwartz: But surrounding that tiny island is an infinite negative ocean of failure. Race conditions, leaked sockets, clock skew flakes, partial rights, and silent memory leaks.

53
00:08:53.390 --> 00:09:04.920
Randal L. Schwartz: Because that negative space is mathematically infinite, no AI lab can ever pre-train or fine-tune a model on every possible mistake.

54
00:09:05.330 --> 00:09:08.760
Randal L. Schwartz: You cannot fit infinity into a training corpus.

55
00:09:09.680 --> 00:09:22.159
Randal L. Schwartz: And that brings us up to the core truth of software engineering. Senior engineering is not knowing how to type syntax. Senior engineering is knowing what not to do.

56
00:09:23.700 --> 00:09:30.670
Randal L. Schwartz: So let's take a quick mental experiment right now. For the next 5 seconds, whatever you do, do not think of a pink elephant.

57
00:09:32.560 --> 00:09:35.540
Randal L. Schwartz: Every single one of us just pictured a pink elephant.

58
00:09:36.970 --> 00:09:56.429
Randal L. Schwartz: Transformer attention mechanisms suffer from the exact same defect. When you write in your agent's MD or cursor rules files, do not mock the parser and do not swallow exceptions. You just loaded those exact forbidden tokens into the model's highest attention weights.

59
00:09:57.620 --> 00:10:02.230
Randal L. Schwartz: Transformers do not process negation like formal Boolean logic.

60
00:10:02.720 --> 00:10:14.389
Randal L. Schwartz: They process statistical token associations. Naming the forbidden antipattern without an escape hatch actually magnetizes the model toward emitting it.

61
00:10:15.580 --> 00:10:28.330
Randal L. Schwartz: And, once the agent is 15 tool calls deep into debugging a tricky test failure, passive wish lists at the top of the prompt suffer severe attention decay and get rationalized away.

62
00:10:30.170 --> 00:10:42.729
Randal L. Schwartz: So how do humans survive an infinite ocean of hazards without reading an infinite library of textbooks? Think about a toddler who reaches out and touches a hot stove burner once.

63
00:10:43.590 --> 00:10:58.899
Randal L. Schwartz: That child does not need a semester of thermodynamics. Their nervous system encodes what neuroscientist Antonio Damasio calls a somatic marker, an immediate visceral physical reflex.

64
00:10:59.090 --> 00:11:01.750
Randal L. Schwartz: They'll never touch that hot stove again.

65
00:11:02.650 --> 00:11:06.160
Randal L. Schwartz: What is 40 years of systems programming experience?

66
00:11:06.330 --> 00:11:08.270
Randal L. Schwartz: It is scar tissue.

67
00:11:08.480 --> 00:11:20.120
Randal L. Schwartz: When I look at a pull request, and my stomach suddenly tightens up, my nervous system is saying, careful, you wrote an async callback like that in 1998, and it paged you at 3 AM.

68
00:11:21.950 --> 00:11:32.240
Randal L. Schwartz: So our central research question became very concrete. How do we give autonomous AI coding agents that exact same protective scar tissue?

69
00:11:34.250 --> 00:11:43.770
Randal L. Schwartz: Our answer is the four-part synthetic scar architecture. Each scar, every scar we codify has the four mandatory vectors.

70
00:11:44.100 --> 00:11:51.129
Randal L. Schwartz: Vector 1 is the wound, the exact runtime exception, memory leak, or silent regression that burned us.

71
00:11:52.620 --> 00:12:08.610
Randal L. Schwartz: Step 2 is the trap. We describe the exact, naive, sunny day tutorial sequence, what we call the 5-6-7-8 statistical impulse, that the model's weights naturally reach for, so the agent catches itself red-handed mid-thought.

72
00:12:08.890 --> 00:12:20.320
Randal L. Schwartz: think about the agent going 1, 2, 3, 4, and then it's wandering off into the woods with 5, 6, 7, 8. That's what we capture as part of the scar.

73
00:12:20.960 --> 00:12:30.330
Randal L. Schwartz: Vector 3 is the circuit breaker, an explicit no. In an epistema… epistem… Epistemic… epistemic, permission slip.

74
00:12:30.630 --> 00:12:43.059
Randal L. Schwartz: The RLHF trains models to be eagle people-pleasers that hate stopping. The circuit breaker gives the model permission to halt and revoke a doomed plan without guilt.

75
00:12:44.380 --> 00:12:58.900
Randal L. Schwartz: And vector 4 is the permanent reflex. Instead of waving a pink elephant, we force concrete orthogonal sequences of actions, EFGH, that routes the agent safely around the cliff edge.

76
00:12:59.420 --> 00:13:06.100
Randal L. Schwartz: So the result becomes 1234EFGH instead of 12345678, boom.

77
00:13:08.230 --> 00:13:22.739
Randal L. Schwartz: Now, let's look at real scars earned in production. In our first case study, an autonomous agent features… finished a feature ticket, ran its test suite, saw green checkmarks, and announced, all units test pass, ready to merge.

78
00:13:23.620 --> 00:13:30.700
Randal L. Schwartz: I didn't write a single line of code. I just looked at the async workflow and asked one Socratic question.

79
00:13:30.890 --> 00:13:41.410
Randal L. Schwartz: Wait, what happens if the caller cancels or shuts down while that async request is still in flight? That's me with my scar, speaking to the agent.

80
00:13:42.670 --> 00:13:57.370
Randal L. Schwartz: The agent froze, traced the resource lifecycle, and realized that canceling mid-request would detonate on disposed state. It immediately wired up clean cancellation teardown, and wrote a red-first regression test.

81
00:13:58.810 --> 00:14:06.590
Randal L. Schwartz: Now… Then we extracted that exact intercession into a four-part async lifecycle scar.

82
00:14:07.050 --> 00:14:17.330
Randal L. Schwartz: I only had to ask that question once. Across the next 200 tickets, the agent asked itself that question before writing a single line of async code.

83
00:14:19.170 --> 00:14:30.910
Randal L. Schwartz: Case Study 2 happens in every language and framework. A ticket asks the agent to write a test verifying our null fallback behavior when a framework lookup returns null inside an application route.

84
00:14:32.480 --> 00:14:46.380
Randal L. Schwartz: A naive AI agent trusts the ticket blindly. It will burn an hour trying to force a null return inside that wrapper, or worse, mock out the framework lookup so the test turns green while testing a scenario that cannot happen.

85
00:14:47.840 --> 00:15:06.899
Randal L. Schwartz: Instead, our contradicted premise scar fired. It forced the agent to read the actual upstream library source code first, proving mathematically that the framework root unconditionally injects a default fallback object. It can never be null there, no matter what you do.

86
00:15:08.010 --> 00:15:19.739
Randal L. Schwartz: Armed with our physical source proof, the agent triggered its circuit breaker, hauled before writing a bogus test, and pivoted to testing bare contexts where the absence is physically real.

87
00:15:22.460 --> 00:15:29.189
Randal L. Schwartz: Now, that leads us to Case Study 3, what we call Senior Planner Mode, not just because I'm a senior, but I'm also a senior.

88
00:15:29.400 --> 00:15:40.720
Randal L. Schwartz: Okay, we realize that many of the worst AI train wrecks happen before coding even starts, because the human's feature spec contains hidden architectural contradictions.

89
00:15:41.230 --> 00:15:51.540
Randal L. Schwartz: When we kick off a multi-ticket initiative, Senior Planner Mode stress tests the proposal against our entire SCAR registry before a single issue is created.

90
00:15:52.660 --> 00:16:03.810
Randal L. Schwartz: The planner is forbidden from guessing API signatures or package behavior from memory. It must query live documentation and inspect actual source boundaries up front.

91
00:16:04.730 --> 00:16:15.149
Randal L. Schwartz: Only after the architecture survives that gauntlet does it emit atomic tickets, each preloaded with the exact failure and variance the coding agent must prove.

92
00:16:17.370 --> 00:16:23.960
Randal L. Schwartz: If you zoom out, almost every developer using AI is climbing a six-stage evolutionary arc.

93
00:16:25.720 --> 00:16:34.210
Randal L. Schwartz: Stage 1 is one-shot prompts and passive wishlist rules. When friction hits, the agent simply rationalizes skipping them.

94
00:16:35.430 --> 00:16:41.570
Randal L. Schwartz: Stage 2 adds version-pinned reference skills and live documentation servers, like Context 7.

95
00:16:41.800 --> 00:16:51.900
Randal L. Schwartz: Reference skills are some awesome for bridging training cutoffs, and they replace the deprecated APIs. They solve the what is the current syntax problem.

96
00:16:53.360 --> 00:16:55.560
Randal L. Schwartz: But Stage 3 and Stage 4.

97
00:16:56.020 --> 00:17:03.879
Randal L. Schwartz: Add procedural workflows and deterministic Dijkstra compiler barriers that halt execution until tests pass.

98
00:17:04.619 --> 00:17:13.379
Randal L. Schwartz: But even at stage 4, once the session ends, the agent still suffers from total amnesia about why a human nudge was needed.

99
00:17:14.940 --> 00:17:18.239
Randal L. Schwartz: And this is what's unique to the SCARS project.

100
00:17:18.619 --> 00:17:36.889
Randal L. Schwartz: We do, in Stage 5 and stage 6, a post-mortem consolidation in phase-bound synthetic scars. Reference skills teach the agent modern APIs. Synthetic scars give it the defensive reflexes to never repeat a production outage.

101
00:17:39.580 --> 00:17:51.669
Randal L. Schwartz: So, how do we enforce those gates without the LLM losing its place? Most people write workflows as sequential checklists. Step 1, write a test. Step 2, write code. Step 3, review.

102
00:17:52.030 --> 00:17:58.000
Randal L. Schwartz: But here's the problem with a sequential list. When step 3 finds a bug and edits the file.

103
00:17:58.390 --> 00:18:02.010
Randal L. Schwartz: The LLM doesn't know that it needs to go back to step one again.

104
00:18:04.170 --> 00:18:10.920
Randal L. Schwartz: We solved this by going back to 1976, to Stuart Feldman's original paper on make.

105
00:18:11.030 --> 00:18:22.150
Randal L. Schwartz: Combined with Edgar Dykstra's guarded commands, we inverted our workflow from a sequential checklist to a declarative makefile-directed graph.

106
00:18:23.620 --> 00:18:39.290
Randal L. Schwartz: Every stage, plan, red test, green test, critic review, commit, depends on the physical artifact on disk, and timestamps for those. You cannot reach commit unless Critic Review MD is newer than every modified source file.

107
00:18:40.640 --> 00:18:48.549
Randal L. Schwartz: The moment an agent touches a single line of source or test code, downstream review stamps automatically become dirty.

108
00:18:49.190 --> 00:18:56.450
Randal L. Schwartz: Loops emerged naturally as mathematical fixed points over file state, with zero prompt amnesia.

109
00:18:57.610 --> 00:19:13.970
Randal L. Schwartz: Inside that directed graph, our golden rule is simple. An operating agent can never serve as the impartial auditor of its own code. If the same context window writes the code and reviews the code, it will rubber stamp its own blind spots.

110
00:19:15.020 --> 00:19:22.080
Randal L. Schwartz: So at the review target, our orchestrator focuses a fresh, ephemeral, adversarial critic subagent.

111
00:19:22.240 --> 00:19:31.729
Randal L. Schwartz: Its system prompt has one mission. Act like the grumpiest senior architect in the building, and try to break the diff across six engineering pillars.

112
00:19:32.590 --> 00:19:49.599
Randal L. Schwartz: Once the PR passes our internal critic and hits GitHub, automated bots like Gemini Code Assist and Copilot chime in. But bots hallucinate, too. If you tell a native, a naive agent, fix the PR comments, it will blindly obey a bad bot's suggestion.

113
00:19:50.150 --> 00:20:04.770
Randal L. Schwartz: So our bot review scar forces the agent to cross-examine every bot thread against the physical codebase invariance, fixing real catches with regression tests, and refuting bot hallucinations with compiler proof.

114
00:20:06.450 --> 00:20:24.690
Randal L. Schwartz: Anyone running autonomous agents on big tickets has hit the context compaction cliff as well. 40 minutes in, the session fills its context window, about 200,000 tokens, and summarizes its history, and instantly forgets all the invariants we verified 20 minutes ago.

115
00:20:25.430 --> 00:20:28.900
Randal L. Schwartz: We solved this with the two classic system ideas.

116
00:20:28.950 --> 00:20:46.499
Randal L. Schwartz: First is the make-fork orchestration. Our primary agent never runs verbose test suites or multi-file greps in its own context. It typically stays under 30,000 tokens as pure supervisor, forking the heavy work to ephemeral sub-agents.

117
00:20:47.340 --> 00:20:51.479
Randal L. Schwartz: Second, I reached back into my 1980s Smalltalk 80 roots.

118
00:20:51.590 --> 00:21:03.459
Randal L. Schwartz: Smalltalk survived power outages using a read-only baseline.sources file, paired with an append-only.changes transaction log, and the condensedChanges operation.

119
00:21:04.290 --> 00:21:09.949
Randal L. Schwartz: We built that exact same Smalltalk 80 image bootloader for AI state.

120
00:21:10.130 --> 00:21:23.349
Randal L. Schwartz: state cards on the disk. Compaction probability drops to zero, and if a session ever restarts, it rehydrates its exact execution frontier in a single read.

121
00:21:25.370 --> 00:21:29.460
Randal L. Schwartz: So how do new scars enter the system? I mentioned a few already in the talk.

122
00:21:29.930 --> 00:21:46.259
Randal L. Schwartz: They enter the system just like in commercial aviation. Every major airliner carries an indestructible black box flight recorder. In our harness, every thought, tool call, and human nudge is written to an immutable JSON-L transcript.

123
00:21:48.210 --> 00:22:02.500
Randal L. Schwartz: When the ticket finally lands, or when we shadow an in-flight session, an independent auditor agent mines that raw flight recorder. It doesn't trust the worker's agent's self-report. It inspects the physical tape.

124
00:22:03.930 --> 00:22:19.549
Randal L. Schwartz: Every time the human had to ask a Socratic question, or a critic caught a blocker, we distill the wound, the trap, the circuit breaker, and the reflance, and bind the new scar directly to the workflow phase where the mistake happened.

125
00:22:21.190 --> 00:22:27.339
Randal L. Schwartz: So when tomorrow's fresh agent wakes up, it doesn't have to relive yesterday's outage.

126
00:22:28.140 --> 00:22:35.470
Randal L. Schwartz: Pain from a single session is compiled in a permanent institutional memory for every future agent.

127
00:22:37.210 --> 00:22:56.290
Randal L. Schwartz: So let's look at the hard empirical numbers. We have now evaluated this architecture across 329, actually it's 331 by this morning, real-world production tickets. 184 in open source libraries and tooling, and 145 in the commercial enterprise monorepos.

128
00:22:57.330 --> 00:23:15.169
Randal L. Schwartz: Along the way, our current registry has grown to 673 codified synthetic scars. 649 of them are public, spanning async teardowns, mock coercion traps, Git work tree isolation, CI triage, and multi-agent governance.

129
00:23:16.420 --> 00:23:30.870
Randal L. Schwartz: So here's the honest scientific breakdown. Across all 329 tickets, scarred code and domain bugs, async teardowns, fake mocks, type traps, stayed at 0.0 repeat progressions.

130
00:23:30.940 --> 00:23:49.320
Randal L. Schwartz: Because they were backed by the physical compiler and TDD barriers. The only 9 repeats, 2.7% overall, were multi-agent timer and polling slips in our orchestrator, where we had to prose… where we had prose scars without the hard harness lock yet.

131
00:23:49.630 --> 00:23:56.539
Randal L. Schwartz: So that contrast actually proves the whole thesis. Every scar ultimately should have a mechanical gate behind it.

132
00:23:58.450 --> 00:24:11.470
Randal L. Schwartz: But you do not need to wait 2 years for the next giant foundation model to ship reliable production software. A scarred workflow with deterministic barriers beats a raw model every single day.

133
00:24:11.770 --> 00:24:24.180
Randal L. Schwartz: And it doesn't even require fancy models. I'm typically using… I had been using Flask 3.8 for a long time, and I'm now just using hydrogen, and it's working fine. And not even on high thinking, it's low thinking.

134
00:24:25.400 --> 00:24:29.539
Randal L. Schwartz: So, I want to be completely honest about where this stands. So, synthetic scars…

135
00:24:29.710 --> 00:24:37.449
Randal L. Schwartz: is in fact a living work in progress. We're still hardening the orchestration boundaries every week as real production edge cases show up.

136
00:24:38.950 --> 00:24:45.819
Randal L. Schwartz: Even in its current form, though, engineers are already getting real value out of their published workflows and public scars.

137
00:24:46.290 --> 00:24:55.260
Randal L. Schwartz: Just this weekend on Reddit, other harness builders and even a non-coding home automation agent picked up on the four-part trap format on the spot.

138
00:24:56.260 --> 00:25:13.360
Randal L. Schwartz: Right now, we're distilling all of this down into what I'm calling the Sourdough Starter Kit, a clean drop-in starter bundle that you can seed into any repository so that your agents grow their own project-specific scar tissue from day one.

139
00:25:14.350 --> 00:25:31.600
Randal L. Schwartz: And once we finish working out the packaging and provenance details, we're going to be setting up a central public scar registry, so that when one team survives a gnarly library or framework trap, everyone else's agents can inherit that reflex without touching the hot stove themselves.

140
00:25:32.950 --> 00:25:46.070
Randal L. Schwartz: Everything we've built so far, the working paper, the live searchable catalog of over 640 public scars, and our published workflow skills, is live right now at syntheticScars.org.

141
00:25:47.500 --> 00:25:54.219
Randal L. Schwartz: Try the published workflow and the four-part SCAR format for your own post-mortems, and tell me where it breaks on your stack.

142
00:25:54.410 --> 00:25:57.440
Randal L. Schwartz: Real-world field reports are how this gets sharper.

143
00:25:58.250 --> 00:26:11.070
Randal L. Schwartz: And reach out to me anytime on GitHub, Blue Sky, X, LinkedIn, all over the socials, usually Randall Schwartz. You can drop me an email at merlin at stonehenge.com, especially if you want to collaborate on the Starter Kit or the Central Registry.

144
00:26:11.480 --> 00:26:21.879
Randal L. Schwartz: And thank you so much for Gabor and all the people at AI Maven. Anyway, so what AI coding traps have you hit lately? Let's go ahead and open the floor.

145
00:26:27.120 --> 00:26:39.300
Gabor Szabo: Well, thanks for the presentation, and I just want to remind everyone that if you would like to have a question, and you don't want to be on the YouTube video, then ask it in the chat.

146
00:26:40.470 --> 00:26:42.630
Gabor Szabo: And,

147
00:26:43.320 --> 00:26:48.689
Gabor Szabo: And it's also fine if you would like to speak up, and you will be in the video in that case.

148
00:26:53.900 --> 00:26:55.000
Gabor Szabo: Let me try to…

149
00:26:55.000 --> 00:26:57.480
Randal L. Schwartz: Questions? Anything?

150
00:26:58.470 --> 00:27:00.989
Gabor Szabo: I'm just sharing the link to the…

151
00:27:01.340 --> 00:27:11.089
Gabor Szabo: syntactic, scars, I shared it in the… Ehh… In the… in the chat.

152
00:27:14.340 --> 00:27:33.820
Gabor Szabo: I personally have the feeling that I need to revisit this and understand it way better way more, and that's one thing. And the other thing that I don't know if you can do that now, or because it's too ad hoc, is to see how to use it, how to work with it.

153
00:27:34.840 --> 00:27:39.570
Randal L. Schwartz: I could probably run one of my open source tickets.

154
00:27:40.670 --> 00:27:46.530
Randal L. Schwartz: But that'll take about 30 to 40 minutes to actually finish, so I don't know that it would be all that great.

155
00:27:46.770 --> 00:27:50.220
Randal L. Schwartz: What I… I can talk you through what it looks like. Yeah.

156
00:27:50.220 --> 00:27:54.670
Gabor Szabo: Yeah, I mean, just, just, yeah, how it looks like, yeah.

157
00:27:55.000 --> 00:28:02.770
Randal L. Schwartz: Yeah, okay, so what it looks like is I open Anti-Gravity, I open in anti-gravity the project, like, block signal.

158
00:28:03.100 --> 00:28:09.030
Randal L. Schwartz: And then, in the chat, I'll say, using Randall's workflow, identify my next task.

159
00:28:09.630 --> 00:28:12.989
Randal L. Schwartz: And the… the workflow is around…

160
00:28:12.990 --> 00:28:16.810
Gabor Szabo: How do you connect… sorry, sorry. How do you connect Randal's workflow?

161
00:28:17.390 --> 00:28:27.570
Randal L. Schwartz: That's what's on the Synthetic Scars site. There's the… there's a gist that contains all the files needed download to install that. The installation instructions are still a bit rough.

162
00:28:27.710 --> 00:28:33.589
Randal L. Schwartz: But it is, you know, if you read the website closely enough, and go to the gist, which is Randall's Gist.

163
00:28:33.690 --> 00:28:36.019
Randal L. Schwartz: That'll be the one to, to check out.

164
00:28:36.500 --> 00:28:39.929
Gabor Szabo: Okay, so I go to the website, and

165
00:28:40.420 --> 00:28:48.359
Gabor Szabo: Actually, you know what? Just a second. I'll share the page, and then we can talk about it. I just have to…

166
00:28:48.500 --> 00:28:49.819
Gabor Szabo: Sure.

167
00:28:50.790 --> 00:28:52.009
Randal L. Schwartz: Good, I'll hit a share.

168
00:28:53.000 --> 00:29:00.790
Randal L. Schwartz: Yeah, the installation is still a little rough, because I'm the only customer right now, so it's pretty easy for me to install it, because it's already installed.

169
00:29:00.890 --> 00:29:13.740
Randal L. Schwartz: But, ultimately, we've had other people, though, start directly from the website and install it or make a modification of it for their workflow. So, we know it's portable enough.

170
00:29:15.370 --> 00:29:17.739
Mateu: Portable to any agent harness, then?

171
00:29:17.740 --> 00:29:18.310
Randal L. Schwartz: Yeah.

172
00:29:18.500 --> 00:29:31.949
Randal L. Schwartz: Right, yeah. Yeah, and we've… I've run some runs with, Claude models as well, and they seem to do reasonably well with it, and… but there are a few specific, Gemini, tricks in the…

173
00:29:32.120 --> 00:29:34.979
Randal L. Schwartz: In there, so we have to figure out how to generalize that.

174
00:29:35.110 --> 00:29:39.649
Randal L. Schwartz: When I say we, I mean me and my lab partner, which is an instance of Gemini somewhere.

175
00:29:44.930 --> 00:29:45.320
Gabor Szabo: Oh.

176
00:29:45.320 --> 00:29:51.260
Mateu: Yeah, I mean, I'm interested in adapting this to my workflow. That sourdough Starter Pack sounds like the place to start.

177
00:29:51.540 --> 00:29:54.949
Randal L. Schwartz: Yeah, we're… it's sort of the next week or two, we should have that done.

178
00:29:56.820 --> 00:29:58.579
Gabor Szabo: Do you see, do you see me?

179
00:29:58.700 --> 00:29:59.190
Gabor Szabo: Yeah.

180
00:29:59.190 --> 00:29:59.940
Randal L. Schwartz: I see.

181
00:30:00.090 --> 00:30:01.560
Randal L. Schwartz: See the… see the webpage.

182
00:30:01.560 --> 00:30:05.130
Gabor Szabo: So you should see your website, the syntax is car architecture, right?

183
00:30:05.390 --> 00:30:06.509
Randal L. Schwartz: Yep, that's me.

184
00:30:06.900 --> 00:30:12.600
Gabor Szabo: Okay… And just a second, I have to stop this, and I remove the other…

185
00:30:12.980 --> 00:30:15.050
Gabor Szabo: the page that I had here.

186
00:30:16.440 --> 00:30:18.139
Gabor Szabo: Okay.

187
00:30:22.860 --> 00:30:38.879
Randal L. Schwartz: Yeah, I've been focusing more on trying to get as many different scars categorized, but it's… I have so many… there's so many scars in there already now that most of the time I'm just saying, continue, continue, continue. There's actually statistics on how many times I've been able to go through an entire ticket run.

188
00:30:39.110 --> 00:30:43.109
Randal L. Schwartz: Without a single nudge, and that's a pretty high number these days.

189
00:30:47.820 --> 00:30:51.669
sean: Yeah, Randall, I can, echo your experience in the…

190
00:30:51.990 --> 00:31:00.189
sean: With the Socratic question. So I'm… I'm driving these AI agents really hard in the operations space.

191
00:31:00.330 --> 00:31:05.130
sean: So, a little bit different, but… I find that, yeah, it's…

192
00:31:05.240 --> 00:31:09.530
sean: You have… you really have to ask those questions,

193
00:31:09.680 --> 00:31:29.520
sean: And that saved a lot of pain, just in the quick review, exactly as you said, what about… what happens if you do this? What about that? And then, that… that caught a lot of things. I've also seen things change over time, though, also, and I've kind of got into a pattern. I'm very, very interested in this.

194
00:31:29.520 --> 00:31:36.059
sean: I'm with a, yeah, four-person startup, so we don't have that scale that you have, and

195
00:31:36.280 --> 00:31:38.899
sean: A lot of, there's not a lot of cross…

196
00:31:39.180 --> 00:31:42.809
sean: you know, person interaction with what we're doing with Clyde, but…

197
00:31:42.810 --> 00:31:43.130
Randal L. Schwartz: Right.

198
00:31:43.130 --> 00:31:48.309
sean: I've settled into a pattern where I now, I'm working with Claude, and I only trust

199
00:31:48.500 --> 00:32:08.060
sean: I only trust, Fable with… to send… to give prompts to, and I'm doing this much, you know, all in a more manual, fashion. And then I ask you to farm out, to… to, like, Opus agents, or sign up the cheapest one possible. And then I have Gemini on the side, and then I have…

200
00:32:08.270 --> 00:32:26.020
sean: So Gemini I use for trading questions, because it's good at that, and then I use the Chinese Kimi… Moonshot Kimi K3 for Linux. I find that it's the best at Linux, better than Fable. So anyway, this is kind of my whole layout, but as we exchanged on LinkedIn.

201
00:32:26.440 --> 00:32:35.880
sean: you know, I'm a former Make guy, I'm very, very interested in this, and we do need to have some, you know, cross-person team…

202
00:32:35.990 --> 00:32:40.349
sean: Knowledge that we, you know, to save tokens.

203
00:32:40.500 --> 00:32:43.409
sean: So yeah, I'm very, very interested in that.

204
00:32:43.770 --> 00:32:44.480
sean: Yeah.

205
00:32:44.480 --> 00:32:45.400
Randal L. Schwartz: Yeah, it's…

206
00:32:45.710 --> 00:33:02.569
Randal L. Schwartz: It's been remarkable, and the evolution has really been a partnership with my Gemini lab assistant, in that I, for a long time, have not ever… I don't write my own skills. I tell the agent what skill I want.

207
00:33:02.740 --> 00:33:08.840
Randal L. Schwartz: And then it writes it better than I can, because it understands itself, if I can use that term loosely.

208
00:33:09.150 --> 00:33:24.229
Randal L. Schwartz: And… I mean, there's a lot of emergent behavior here, like the fact that it can read the flight recorder of another session and extract where I nudged and why I had to nudge it and what the agent was thinking requires almost an emergent theory of mind.

209
00:33:24.760 --> 00:33:36.920
Randal L. Schwartz: But somehow it manages to do it. It works, and it works over and over again. So it's really… to me, it's… this could not have been possible 6 months ago. What I'm doing today could not have been done 6 months ago, because they weren't…

210
00:33:36.920 --> 00:33:37.340
sean: Alright.

211
00:33:37.340 --> 00:33:44.609
Randal L. Schwartz: that self-aware, if you can… again, using humanizing terms for that, but they weren't that… they weren't that…

212
00:33:44.830 --> 00:33:47.970
Randal L. Schwartz: They didn't have theory of mind, to this degree.

213
00:33:48.200 --> 00:33:56.649
sean: Yeah, and I want to add one thing. One of my biggest frustrations is actually the writing, and it seems that the… so, Fable is way too verbose.

214
00:33:56.800 --> 00:34:01.520
sean: And so, I've tried to develop skills, and I'm thinking of…

215
00:34:01.690 --> 00:34:20.090
sean: you know, saying, hey, Sonnet, go rewrite what Fable wrote, and make it, you know, 60% shorter. But, it's quite… it is quite interesting to try to teach it how to communicate better, and it's like, no, I don't care about the history, I just want the state. It's go, like, whoa.

216
00:34:20.090 --> 00:34:20.440
Randal L. Schwartz: I have…

217
00:34:20.440 --> 00:34:26.879
sean: this on this date, and blah blah blah, and then you did this, and then this happened, and then John changed this, and I'm like, you know.

218
00:34:26.889 --> 00:34:27.229
Randal L. Schwartz: I…

219
00:34:27.230 --> 00:34:28.129
sean: Tell me what it is.

220
00:34:28.130 --> 00:34:39.240
Randal L. Schwartz: I have… I have an advantage in that my voice is well captured. In particular, I learned the trick in my press agent, press relations agent.

221
00:34:39.350 --> 00:34:45.990
Randal L. Schwartz: to say, Write like the author of Learning Pearl and Intermediate Pearl.

222
00:34:46.770 --> 00:34:56.020
Randal L. Schwartz: And it has studied all that. It has… that's part of the stuff it absorbed. So it knows exactly what I sound like when I'm writing long tutorials.

223
00:34:56.199 --> 00:35:03.840
Randal L. Schwartz: And so, I'm able to draw on that as my body of knowledge, and it's just a few keywords in the prompt.

224
00:35:03.840 --> 00:35:06.869
David: You literally are embedded in the training data already.

225
00:35:06.870 --> 00:35:11.699
Randal L. Schwartz: Exactly, exactly. It's true, it's true.

226
00:35:12.320 --> 00:35:12.780
Randal L. Schwartz: No, well…

227
00:35:13.300 --> 00:35:14.090
sean: No, sir, go ahead.

228
00:35:14.090 --> 00:35:22.119
Randal L. Schwartz: Read the, read the opening of the paper. The opening of the paper, actually, if I can pull it up here. We've got a couple quotes there.

229
00:35:22.370 --> 00:35:23.140
Randal L. Schwartz: See how do we get.

230
00:35:23.140 --> 00:35:27.630
David: I'm gonna ask Opus today if you could make this more Schwartzian and see if he knows what I'm talking about.

231
00:35:27.630 --> 00:35:29.500
Randal L. Schwartz: Oh, speaking of…

232
00:35:29.500 --> 00:35:31.399
Gabor Szabo: Remind everyone we are still recording.

233
00:35:31.780 --> 00:35:33.440
Randal L. Schwartz: Oh, yeah, sorry.

234
00:35:33.440 --> 00:35:36.630
Gabor Szabo: No problem, it's just I wanted to remind you.

235
00:35:36.630 --> 00:35:41.469
Randal L. Schwartz: Any more questions for me? Any more, anything that wasn't clear in the presentation?

236
00:35:42.310 --> 00:35:44.690
Randal L. Schwartz: And was the presentation about the right?

237
00:35:44.880 --> 00:35:52.210
Randal L. Schwartz: Pace? Do you think it was, you know, covering a thing? Was there too much covered in the 35 minutes or so that I took?

238
00:35:53.620 --> 00:35:54.609
sean: No, not at all.

239
00:35:54.610 --> 00:36:00.669
Gabor Szabo: Let's see, yeah. I think, okay, we can further discuss it afterwards, right? After the recording.

240
00:36:01.150 --> 00:36:01.700
Randal L. Schwartz: Pugh.

241
00:36:02.060 --> 00:36:10.979
Gabor Szabo: Okay, so let me, let me finish the recording, and then we can stay on, everyone here, and we can continue the discussion without,

242
00:36:11.270 --> 00:36:15.209
Gabor Szabo: scary part of being recorded and published, but…

243
00:36:15.210 --> 00:36:17.019
Randal L. Schwartz: On YouTube?

244
00:36:17.020 --> 00:36:35.310
Gabor Szabo: Thank you very much for this presentation, and thanks everyone for being here and asking questions, or not asking questions, doesn't really matter. In this case, I thank you both. And thanks everyone who's watching the video, and please don't forget to like the video, and follow the channel, and then remind you

245
00:36:35.310 --> 00:36:41.990
Gabor Szabo: And I'd like to remind you that below the video, you will find links to future events and to various things,

246
00:36:42.330 --> 00:36:49.790
Gabor Szabo: about this event, and so on. So, thank you very much, and, goodbye!

