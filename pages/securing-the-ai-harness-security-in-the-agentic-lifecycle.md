---
title: "Securing the AI Harness: Security in the Agentic Lifecycle"
timestamp: 2026-11-17T12:30:01
author:
published: true
description:
tags:
---

As AI agents become increasingly integrated into software development, we are not only engineering models, but the harnesses around them - their tools, context, credentials, and interactions with external systems. This creates new security surfaces where sensitive data can unintentionally flow from trusted environments into model context, telemetry, logs, or external services. This lecture explores the security vulnerabilities that emerge within harness engineering, how to reason about sensitive-data flow across the agentic lifecycle, and practical best practices for reducing these risks. It will also introduce Weir, an open-source tool for deterministically testing and monitoring credential flows in AI agents.

## Bio

[Ido Goldstein](https://www.linkedin.com/in/ido-goldstein-9ba87b152/)

Software Engineer from big tech (Meta), Cyber startups (Token, Ironscales) and IDF. I innovate security in Agentic Development Lifecycle.

## Length

40 minutes

* [weir](https://github.com/ralis-ai/weir)


{% youtube id="RM66ljIJwgE" file="2026-11-17-Securing-the-AI-Harness-Security-in-the-Agentic-Lifecycle-with-Ido-Goldstein.mp4" %}


## Transcript

1
00:00:01.970 --> 00:00:13.930
Gabor Szabo: So, hello and welcome to the Code Maven channel, and specifically to our first, actually, AI Maven meeting. Hello, Ido, thank you for agreeing to give this presentation.

2
00:00:13.930 --> 00:00:22.889
Gabor Szabo: And thanks everyone who joined us in this meeting today. You have the option later on to ask questions in the chat.

3
00:00:22.900 --> 00:00:41.680
Gabor Szabo: those people who are watching the video on YouTube, below the video, you are going to find a link to the future events, so you can check that out. You will also find a link to this event and various things about this event, maybe even the slides, we'll figure it out.

4
00:00:41.680 --> 00:00:47.280
Gabor Szabo: And if you don't want to share links, then those will be there below the video.

5
00:00:47.280 --> 00:00:54.960
Gabor Szabo: And don't forget to like the video, and follow the channel, and I hope to see you at one of our future events.

6
00:00:54.970 --> 00:01:02.910
Gabor Szabo: And now, again, thank you, Ido for agreeing this, to this presentation, and the floor is yours.

7
00:01:03.180 --> 00:01:21.469
Ido Goldstein: Yeah, thank you, Gabor. So, hey guys, I'm going to speak today about agent harnessing. I'm going to walk through some examples, some real examples, and, see how we can, secure it.

8
00:01:21.900 --> 00:01:25.280
Ido Goldstein: So, a little bit about myself.

9
00:01:25.420 --> 00:01:28.260
Ido Goldstein: Those who know, those who do not.

10
00:01:28.290 --> 00:01:46.539
Ido Goldstein: I have a track record for the last 7 years in software, from big companies to small cyber ones. And, today, I try to, help organization end-to-end pipelines with security frameworks, so…

11
00:01:46.540 --> 00:01:50.449
Ido Goldstein: In general, in general?

12
00:01:50.500 --> 00:02:01.740
Ido Goldstein: I try to help them, when the pain is most appearing in regard to security.

13
00:02:01.770 --> 00:02:17.589
Ido Goldstein: We're going to walk, some examples, and hopefully you can, you know, take it to your own, and implement it, the practices that I will, show, and, it will be worthwhile.

14
00:02:18.060 --> 00:02:22.450
Ido Goldstein: So, just to,

15
00:02:22.640 --> 00:02:32.670
Ido Goldstein: everyone to be aligned, so, we're going to, discover and, align what RTMS engineering really is.

16
00:02:32.670 --> 00:02:44.270
Ido Goldstein: We're going to follow some example in census data in Agentic lifecycle, in comparison with the software lifecycle that was, appearing

17
00:02:44.390 --> 00:02:50.760
Ido Goldstein: All of our lifetime, probably. That, has been changed, in the last,

18
00:02:50.900 --> 00:02:55.119
Ido Goldstein: Year, year or so, maybe 2 years, depending who you ask.

19
00:02:55.250 --> 00:03:13.839
Ido Goldstein: And, we apply… we're going to, apply some, my own, practices, and see, how we can, each of the ones that, the leakage that we are going to discuss.

20
00:03:13.840 --> 00:03:15.970
Ido Goldstein: How we are going to prevent it, or…

21
00:03:15.970 --> 00:03:31.689
Ido Goldstein: at the very least, minimize the attention that it will cause in production systems. We're going to walk through, a real scenario, real-world scenario that I found, the CVE.

22
00:03:31.750 --> 00:03:44.689
Ido Goldstein: And we're going to discuss determinism, how we can make determinism in the Agentic lifecycle that is, by design, not deterministic.

23
00:03:44.790 --> 00:03:59.300
Ido Goldstein: Later, I will discuss the habits that I practice, and I will discuss my own open-source personal tool that does some of those tasks, and represent it, which his name is Weir.

24
00:03:59.900 --> 00:04:05.850
Ido Goldstein: So, let's get to it. So what is harness engineering? Harness Engineering…

25
00:04:05.850 --> 00:04:16.469
Gabor Szabo: Sorry, maybe you want to switch to presentation mode here, because in the slides, at least I see the regular editing part of it.

26
00:04:16.940 --> 00:04:18.450
Ido Goldstein: Alright,

27
00:04:18.459 --> 00:04:21.089
Gabor Szabo: At the bottom, you can see the link,

28
00:04:22.360 --> 00:04:22.960
Ido Goldstein: Here?

29
00:04:22.960 --> 00:04:30.139
Gabor Szabo: No, actually, yeah, I think the first leftmost button is the one. I'm not that familiar at the bottom.

30
00:04:31.250 --> 00:04:31.890
Ido Goldstein: That one?

31
00:04:31.890 --> 00:04:34.170
Gabor Szabo: No, no, no, to the right to it.

32
00:04:35.220 --> 00:04:35.819
Gabor Szabo: I see.

33
00:04:37.060 --> 00:04:46.429
Ido Goldstein: I think I pressed that, so, it's a representation mode, so… I'm seeing it in my own screen, like, in a big…

34
00:04:46.830 --> 00:04:48.260
Ido Goldstein: Fullscreen.

35
00:04:48.260 --> 00:04:54.400
Gabor Szabo: No, I see… I see the editing mode.

36
00:04:55.070 --> 00:04:57.520
Ido Goldstein: Oh, alright, so…

37
00:04:59.750 --> 00:05:01.559
Gabor Szabo: Maybe you need to share a different screen.

38
00:05:01.560 --> 00:05:03.149
Ido Goldstein: Yeah, yeah, probably.

39
00:05:10.300 --> 00:05:12.720
Ido Goldstein: Let's try to do it that way, maybe?

40
00:05:13.730 --> 00:05:15.959
Gabor Szabo: Yeah, yes, yes, thank you.

41
00:05:15.960 --> 00:05:21.960
Ido Goldstein: Alright, yeah, for sure. So, what is hardness engineering? Real quick?

42
00:05:22.090 --> 00:05:29.959
Ido Goldstein: Harness engineering, those who know, those who do not, is essentially a model and a harness.

43
00:05:30.220 --> 00:05:46.440
Ido Goldstein: Harness is just everything that is not the model. So, think of it as a… how do you describe an automobile? So, you have the engine, and you have everything around it. So, you can have the same engine, the same model, but a different

44
00:05:46.440 --> 00:05:58.299
Ido Goldstein: car keys, or different… different set of tools, above it, which creates a different car. So this is… this is the… the, like, absurd definition of an agent.

45
00:06:00.360 --> 00:06:16.860
Ido Goldstein: there are six things that we are now building in that harness. So, if we are delving into that core and trying to understand how it assembles, so we have a context, we have instructions, we have an agent.md, we have tools.

46
00:06:16.860 --> 00:06:30.200
Ido Goldstein: the tools can be shell, MSP, MCPs, browser, API, anything that the model can have access to. We have identity, control, state, and feedback.

47
00:06:31.390 --> 00:06:34.059
Ido Goldstein: Most of it, guide.

48
00:06:34.280 --> 00:06:47.229
Ido Goldstein: But, we only, then sensor. So, if we accommodate all of those, the things that, accommodate the harness.

49
00:06:47.350 --> 00:06:52.660
Ido Goldstein: We have a feed forward and a feedback, so we can steer the agent.

50
00:06:52.970 --> 00:07:10.799
Ido Goldstein: And we can check the results. So, this is the concept of harnessing, right? So, we have a set of tools, we have a set of skills, and then we have a feedback. The feedback can be put as a hook. Let's say the lens, the test, the reviews, the monitoring. Each

51
00:07:10.800 --> 00:07:17.860
Ido Goldstein: Happens to replace a human part that was done for in the previous, like, the past.

52
00:07:20.090 --> 00:07:37.269
Ido Goldstein: So why does it matter? Because right now, we are entering an age that everything is AI. Everything is being written by AI, nothing is written by hand. This is from earlier this year, even more so today.

53
00:07:37.670 --> 00:07:55.259
Ido Goldstein: So, what is the SDLC, and how, it actually created the ADLC, the Agentic Development Lifecycle? So, in the old days, if you remember a year ago or two, there were tickets.

54
00:07:55.260 --> 00:08:00.359
Ido Goldstein: There were human writing, and they were reviewing.

55
00:08:00.360 --> 00:08:13.240
Ido Goldstein: Each of it has been replaced with agent right now, so we… the stuff that we had earlier, need to plan, to build, to review, now have been replaced with agents.

56
00:08:14.220 --> 00:08:20.649
Ido Goldstein: There are 3 facts right now. The agent acts as you.

57
00:08:20.890 --> 00:08:31.680
Ido Goldstein: every text is actually a surface, we'll get it in a minute, later on, like, how… how is that a problem? And…

58
00:08:31.680 --> 00:08:41.499
Ido Goldstein: every step in… along the way is being saved, so transceived, we have traces. Each of it, actually is being saved right now and monitored.

59
00:08:41.710 --> 00:08:48.099
Ido Goldstein: We need to know that because, if the agent can read it, it can be steered to send

60
00:08:48.210 --> 00:08:56.469
Ido Goldstein: a text, or a data. So, every data, it's a plane. A different plane that now needs to be broken down.

61
00:08:56.800 --> 00:09:00.339
Ido Goldstein: Save every data in the Agentic lifecycle.

62
00:09:00.850 --> 00:09:13.149
Ido Goldstein: So, sensitive data is not just secrets. We'll get to it in a minute, but, think of it as what you have to, safeguard.

63
00:09:13.260 --> 00:09:17.359
Ido Goldstein: When you are pushing stuff that's beyond your comprehension.

64
00:09:17.790 --> 00:09:25.799
Ido Goldstein: Right? I want you to… if you take something with those earlier slides, I want you to take this, that

65
00:09:26.420 --> 00:09:44.749
Ido Goldstein: we don't need just to safeguard our passwords, or API, or SSH keys. We need to safeguard our… every sensitive data. Personal information, billing records, health. In each sector, it can be something different.

66
00:09:44.780 --> 00:09:57.219
Ido Goldstein: But each data right now, it can be leveraged in the next exposure. So we need to understand that it's more than a secret, it's the data itself.

67
00:09:57.290 --> 00:10:11.069
Ido Goldstein: And there are four stages that we're going to walk through, and very common CV that I chose that I think emphasizes that point. So, each and everyone will see a wreath.

68
00:10:11.180 --> 00:10:24.549
Ido Goldstein: We're going to see how it, it's been kept. We're going to see, what's scented, if it's vehicle, or by a tool, sorry, or by other, sort of mean of action.

69
00:10:24.640 --> 00:10:32.369
Ido Goldstein: And we're going to see how it's been read, and recorded in the telemetry afterwards.

70
00:10:32.670 --> 00:10:35.839
Ido Goldstein: So, I think the most

71
00:10:35.900 --> 00:10:44.190
Ido Goldstein: common exposure, was the change of supply, the… it's been called the annexed singularity, so…

72
00:10:44.190 --> 00:11:06.159
Ido Goldstein: There is a thing with CVEs that, try to sell it, so it's… they're giving each and the CVE that, broke the internet, some, how do you say, acquisite name. So this one was called the NX Singularity, so there is an NX, library that sits under NPM.

73
00:11:06.230 --> 00:11:15.170
Ido Goldstein: attacker managed to, actually, forge it for about 4 hours,

74
00:11:15.260 --> 00:11:27.150
Ido Goldstein: And it, causes, over 2,000 secret leak and, machines. The Poison NPM, had run an AI

75
00:11:27.480 --> 00:11:34.570
Ido Goldstein: in a CLI, and the checks were actually turned off. How did it manage to do that? Because

76
00:11:34.570 --> 00:11:50.210
Ido Goldstein: it was the… the… the data, the… the… the… all the keys that, actually, build upon it, right? The… all who uses the NX, library, actually, made his

77
00:11:50.370 --> 00:11:53.880
Ido Goldstein: data into a public Gita bridge when the attacker uses it.

78
00:11:55.880 --> 00:12:06.709
Ido Goldstein: this is, this is very, very common, one of the most common exposures that happened, in the, recent years. We…

79
00:12:06.710 --> 00:12:26.539
Ido Goldstein: an attacker managed to manipulate the harness by writing a ticket. So, think of it like that. You are pushing a PR, and you are deliberately writing a… you can do it in a public repo as well, and by the text that it pushes, in the ticket, it managed to

80
00:12:26.840 --> 00:12:35.930
Ido Goldstein: Actually, affect… an agent that was running on top of the PR, right? You have code reviewers.

81
00:12:35.960 --> 00:12:45.680
Ido Goldstein: You have all those bots that read your tasks and create tickets. So, he managed to affect that one of those agents.

82
00:12:45.680 --> 00:12:56.980
Ido Goldstein: and causes a zero day in all those environments. So it was touched by Cloud Call, by Gemini, by Codex. It was given the highest score.

83
00:12:57.280 --> 00:13:04.369
Ido Goldstein: In CVEs, which is, tells, telling, how, how vulnerable WASIS.

84
00:13:04.720 --> 00:13:19.019
Ido Goldstein: There was another outreach researched by Apiro, which is a software security company that, just, did something similar with a password.

85
00:13:19.160 --> 00:13:29.199
Ido Goldstein: Also, there is a Salesforce attack that causes the CRM to… just loses the MFA.

86
00:13:29.350 --> 00:13:49.010
Ido Goldstein: There is also some degree of CVEs that happened, and the most famous one is the EcoLink. Ecolic was the first one in late 2025, that, actually got a new co-pilot agent.

87
00:13:49.050 --> 00:13:52.119
Ido Goldstein: To, actually…

88
00:13:52.530 --> 00:14:08.649
Ido Goldstein: go into public with sensitive data that was, sending via an email. So, it also happened, if you here… if you had, a couple days ago, but I wasn't able to put it into the presentation, sadly.

89
00:14:08.650 --> 00:14:16.350
Ido Goldstein: But the Microsoft 365, was going over a victim's emails.

90
00:14:16.350 --> 00:14:27.670
Ido Goldstein: And because one of the emails, contained, instruction that was led to believe, part of the email, it led to,

91
00:14:27.880 --> 00:14:42.119
Ido Goldstein: a massive amount of data from the victim's email box to be sent to the attacker. This was the first one, and it represents a pattern here, right? So,

92
00:14:42.470 --> 00:14:50.730
Ido Goldstein: Pixel leak also was the same. So, it represents a pattern. So, AI-assisted comments and AI-assisted code.

93
00:14:50.990 --> 00:15:02.949
Ido Goldstein: tend to leak and causes a damage, right? That damage is, more often than not, is not being monitored by the auditors.

94
00:15:03.300 --> 00:15:04.290
Ido Goldstein: So…

95
00:15:04.400 --> 00:15:11.840
Ido Goldstein: what we have to do, right? So, it's not all dark and grovel. We can do something about it.

96
00:15:11.900 --> 00:15:26.850
Ido Goldstein: So, in my mind, we always have to look for the little trifecta. So, we have to look for access to private data. Let's say that email box, right? It contains a private data.

97
00:15:26.850 --> 00:15:36.570
Ido Goldstein: we have to look for exposure to untrusted texts, let's say, in the common PR, right? We open… we allow everyone to open a new PR, so…

98
00:15:36.570 --> 00:15:50.379
Ido Goldstein: We can't allow a direct contact with an agent to someone that just opened a new PR. We don't know who is it, and it can manipulate the agent to accept or,

99
00:15:50.680 --> 00:16:02.309
Ido Goldstein: actually change how it, received its decisions. So, and we don't… we don't want to, practice a way to send the data out.

100
00:16:02.500 --> 00:16:08.979
Ido Goldstein: All three together, it would probably fit into an exploitation path.

101
00:16:09.570 --> 00:16:17.140
Ido Goldstein: So, we need to practice find it, we have to classify it, and we have to handle it. And this is what…

102
00:16:17.520 --> 00:16:21.530
Ido Goldstein: the best practices, in my mind, are.

103
00:16:21.630 --> 00:16:39.399
Ido Goldstein: we have to look for what agent can reach. We have to look for the machine, the repo, the telemetry, the CI. We need to know to, actually to map, how the agent reaches the data.

104
00:16:39.630 --> 00:16:44.260
Ido Goldstein: Before it may go, because,

105
00:16:44.640 --> 00:16:55.659
Ido Goldstein: every single data that we send, every single credential that we have… that the agent can access to. We need to practice one common path, maybe two.

106
00:16:55.660 --> 00:17:09.949
Ido Goldstein: and think least privilege in that sense. So, if we have a password, we have to only, allow one service to go through it, and to… to… to actually, touches it.

107
00:17:10.000 --> 00:17:26.580
Ido Goldstein: If we have, public records, we can allow them to go anywhere. Why? Because it's public. It's, like, something that is not, can be put to use against us. Again, restricted data. We have to, know

108
00:17:26.579 --> 00:17:34.120
Ido Goldstein: where things are mapped. We have to classify them. So the rule… the rule of thumb here, is this value.

109
00:17:34.120 --> 00:17:48.399
Ido Goldstein: somewhere isn't allowed, right? So, this is always had to be in your mind, and we need to handle it. We have 5 ways that we can practice it.

110
00:17:48.570 --> 00:17:58.119
Ido Goldstein: in my mind, we have to minimize the load, right? So, keep it always out of the agent context, and the harnesses.

111
00:17:58.150 --> 00:18:11.249
Ido Goldstein: both ways. We're also going to touch it. We have to… we can mask it, right? Before it ends the context, if it ended the context, we have to replace it with a stub or something that is not vulnerable.

112
00:18:11.380 --> 00:18:30.320
Ido Goldstein: Give it a decoy, so it will alarm and give us the status if some credential in the field happened to be found. And we have to shrink the attack surface. We need to practice list privilege. And of course, if…

113
00:18:30.320 --> 00:18:32.719
Ido Goldstein: To apply expiry.

114
00:18:33.700 --> 00:18:37.970
Ido Goldstein: So this is, like, the policy, this is what I just said.

115
00:18:37.990 --> 00:18:49.149
Ido Goldstein: The fourth stage, right? So, I'm going to discuss, like, the CV that I personally found. After all this,

116
00:18:49.150 --> 00:19:02.040
Ido Goldstein: practices that I've made, how I actually end up with, MLflow, which is one of the largest repos out there with 20 million downloads a month. How did I manage to cut

117
00:19:02.120 --> 00:19:05.820
Ido Goldstein: A new credential, out there.

118
00:19:06.850 --> 00:19:12.030
Ido Goldstein: So, the MLflow had a autologger that, actually,

119
00:19:12.630 --> 00:19:27.759
Ido Goldstein: take the crew AI, who's not familiar, Crew AI is a common agent that, is being put to use to orchestrate agents' work, and also it's a big, big, big, very big public repo.

120
00:19:27.920 --> 00:19:38.730
Ido Goldstein: And, there is an autogo that trunks the tracing for every Core AI agent.

121
00:19:39.510 --> 00:19:47.829
Ido Goldstein: So, the key was just playing the record, right? The… the auto logo was outside to MFflow.

122
00:19:48.060 --> 00:19:53.500
Ido Goldstein: And what it did, actually, it got over the span.

123
00:19:54.050 --> 00:19:57.620
Ido Goldstein: took the LLM object, it was a Pythonic object.

124
00:19:57.760 --> 00:20:04.209
Ido Goldstein: And just put in plain text, the API key that

125
00:20:04.340 --> 00:20:12.499
Ido Goldstein: in there. So, it's, it's those who are coming from software, it's, called a serialization of object.

126
00:20:12.810 --> 00:20:21.279
Ido Goldstein: So, the key itself was just out there for about 20… 47 releases.

127
00:20:22.000 --> 00:20:36.259
Ido Goldstein: It's a very, very long time. 22 months that it was out there, in the field, waiting for an attacker, maybe, been put to use.

128
00:20:36.440 --> 00:20:38.360
Ido Goldstein: So,

129
00:20:38.820 --> 00:20:57.610
Ido Goldstein: the key was used to, to, correctly in the agent, but the stored trace actually wasn't monitored. So, every OTLP, OTLP is, the format of the traces, actually dashboard that same credential.

130
00:20:57.810 --> 00:20:58.890
Ido Goldstein: So…

131
00:20:59.730 --> 00:21:10.709
Ido Goldstein: how did I found it? I didn't read the code, I wasn't, you know, familiar that much with Emmanuel, I just knew it by hand. So I just,

132
00:21:10.710 --> 00:21:21.049
Ido Goldstein: actually practice the three factors that I just experienced. So, I planted a unique fakie, right, that I knew.

133
00:21:21.160 --> 00:21:40.110
Ido Goldstein: I initiate the agent flow, and then I just search for that fake key. It's that simple, guys. Just, go and be verbatim, how can a secret be found in the telemetry that it exposes?

134
00:21:42.110 --> 00:22:01.269
Ido Goldstein: So, of course, how MFlow reached that conclusion, it was in version 3.17, it rotated the keys, and of course, it purged all the recent values that were… was before it.

135
00:22:01.620 --> 00:22:02.700
Ido Goldstein: So…

136
00:22:02.760 --> 00:22:18.610
Ido Goldstein: the common practice that you need to put in your mind, that your dependencies… right now, each of us have dependencies. We use NPM, we use Python, we use PIP. Your dependencies decide where your data goes, not your flows.

137
00:22:18.610 --> 00:22:25.469
Ido Goldstein: Not just the code that you practice, your dependencies, your whole software as a paradigm.

138
00:22:25.600 --> 00:22:31.309
Ido Goldstein: So you have to put it in your mind that you always have to be exposed.

139
00:22:31.420 --> 00:22:35.449
Ido Goldstein: And monitor what your dependencies are.

140
00:22:36.280 --> 00:22:49.619
Ido Goldstein: So, how can we build? We… I think we agree that there is a problem here, right? We have a massive amount of CVEs, we have,

141
00:22:50.060 --> 00:22:59.629
Ido Goldstein: No way to know where we have that reflector, so how we can make sense out of it, and build toward determinism.

142
00:23:00.630 --> 00:23:18.349
Ido Goldstein: So, first and foremost, the common practice with identity. So, we have not privileged roles, overprivileged roles, we have, scoped identities, we have scoped access, we control the inputs, right? We don't need to,

143
00:23:18.350 --> 00:23:34.810
Ido Goldstein: treat every ticket as an exposure, but we do need to know if a common vulnerability has been accessed into those tickets. It can be emails, it can be agent, MD, everything is untrusted.

144
00:23:35.230 --> 00:23:42.470
Ido Goldstein: And the tools need to be enforced. We need to know, in each flow, where the data might flow.

145
00:23:43.650 --> 00:23:50.480
Ido Goldstein: And it works for the ingress, right? So, never…

146
00:23:50.830 --> 00:24:08.259
Ido Goldstein: run all the trees at once. Never do the… also the read, the record, and the allow list node, as one. So just split the job, you can add a human in the loop. It's not a very common practice, but you can do it.

147
00:24:08.560 --> 00:24:22.840
Ido Goldstein: you can decide if you go to an allow list, and you can have a retention of your traces. So you always have to look back into what happened and what caused the analysis.

148
00:24:23.320 --> 00:24:36.440
Ido Goldstein: So, the catch here is to expect the same input each and every time. And if you get a different advert, you know, some… maybe the God rate wasn't good enough.

149
00:24:36.890 --> 00:24:53.810
Ido Goldstein: So, today, just, it, it works out, a couple of weeks ago, that, for Opus, 5.5 is, you know, the hardest to, to, to jailbreak, but even that model is still not safe.

150
00:24:53.810 --> 00:24:59.770
Ido Goldstein: It has, less than 5% ability to breaks.

151
00:24:59.770 --> 00:25:14.110
Ido Goldstein: But if the attacker is being exposed with the response, those percentages go to almost 42%. That a lot, that is a lot, so…

152
00:25:14.110 --> 00:25:23.820
Ido Goldstein: You always have to guard with how your system is responding, because then you are, you know, 10 times more exposed.

153
00:25:25.010 --> 00:25:43.300
Ido Goldstein: So, all the inferential one that revolve around agents, can be put with practices to guard against. So, if you have a system prompt, LLM fit or LLM jazz, you can actually apply hooks.

154
00:25:43.300 --> 00:25:47.720
Ido Goldstein: You can have a allow list, you have proxy,

155
00:25:47.930 --> 00:26:02.890
Ido Goldstein: I don't think we have much time, so I'll just go for it. So the main niches, the model decides what you do, but the code decides what's allowed. So, this one you can configure. You can configure who is allowed to do what.

156
00:26:03.010 --> 00:26:16.039
Ido Goldstein: And the four habits that I practice is don't French, don't show, don't send, and have it proved. This is the most, I think, important in this session. You have to have it proved, because

157
00:26:16.110 --> 00:26:35.180
Ido Goldstein: And if you don't, you don't have the determinism that you have to actually build on it. So, you can block the commands, but the print keys whatsoever, so no agent can actually print keys.

158
00:26:35.240 --> 00:26:40.009
Ido Goldstein: You can just repress it with another name.

159
00:26:40.140 --> 00:26:49.149
Ido Goldstein: If you have the credential inside your hooks, inside your agent.

160
00:26:49.150 --> 00:27:03.699
Ido Goldstein: You can replace it with a stop that is not harmful, right? You can stripe the keys before… just before the model sees them. Cloud Code is already open to that sort of features.

161
00:27:03.850 --> 00:27:18.559
Ido Goldstein: And you can have the… given the agent a decoy in CI. So, each time the pattern of your credential is being put to use, you can see if something broke out.

162
00:27:19.690 --> 00:27:32.629
Ido Goldstein: So, this is, I think, the most important one, so you can actually prove it. So, if a plant keys actually managed to go out, you can…

163
00:27:32.930 --> 00:27:39.570
Ido Goldstein: Then check if your system is… how do you say?

164
00:27:39.730 --> 00:27:43.440
Ido Goldstein: It's God is, tight enough.

165
00:27:43.850 --> 00:27:46.130
Ido Goldstein: So,

166
00:27:46.600 --> 00:28:06.950
Ido Goldstein: This is something that I also managed to do, to actually provide that sort of insight, if you have a leak, if you have something that is clean, and most importantly, if something is not immeasurable. So, you have to have an indication that it's incomplete.

167
00:28:07.040 --> 00:28:26.980
Ido Goldstein: So, this is what I do with my tool, Weir. Those who don't know, Weir is, like, a small dam that allows you to measure the flow of water, so the key concept here is to keep your flow intid, but, just, not interrupt the flow.

168
00:28:27.240 --> 00:28:31.329
Ido Goldstein: But, you know, just to keep it going.

169
00:28:31.750 --> 00:28:40.920
Ido Goldstein: So, in that tool that I've built, on top of the CV that I found, is, just,

170
00:28:41.150 --> 00:28:49.790
Ido Goldstein: see if your system leaks API keys. So, it's very easy to install.

171
00:28:49.790 --> 00:29:09.410
Ido Goldstein: And it provides all the features that I actually had represented here. You can have here, for example, a provider model by OpenAI, and then it can be replaced before the agent ever reaches it.

172
00:29:09.530 --> 00:29:19.019
Ido Goldstein: And of course, it also runs in CI, so it fails the PR if it found any of the planted API keys.

173
00:29:19.470 --> 00:29:36.480
Ido Goldstein: And it is also a small hook. This is a picture from Cloud Code, so, you can see, if your harness, has reached to some untrusted data that you can define it.

174
00:29:36.860 --> 00:29:49.009
Ido Goldstein: Of course, it has some limits. It only runs in the LAN, and it's only… the only isolation that is provided is on Linux. It's more to be developed.

175
00:29:49.380 --> 00:30:06.819
Ido Goldstein: And I think the second part of it, we only discussed here cyber and cyber behavior, but I think the more pressing matter is how the behavior can be changed. This is a metaphor, another one, but just keep it in mind.

176
00:30:06.820 --> 00:30:10.260
Ido Goldstein: That, we discuss only if a key leave, but…

177
00:30:10.330 --> 00:30:14.510
Ido Goldstein: How can we, safeguard the behavior?

178
00:30:14.760 --> 00:30:16.880
Ido Goldstein: Just, keep it in mind.

179
00:30:17.130 --> 00:30:34.329
Ido Goldstein: And again, just to, think of these three topics, so you have, a thing, not necessarily VIO, you know, weird, but you have to, have, a model to classify.

180
00:30:34.330 --> 00:30:38.109
Ido Goldstein: where the data may go, you need to answer that question.

181
00:30:38.110 --> 00:30:43.369
Ido Goldstein: You have to… you have to build boundaries in your code, not in your prompts.

182
00:30:43.760 --> 00:30:50.310
Ido Goldstein: And always, always, always have a way to prove it, so test the flow on every PR.

183
00:30:50.820 --> 00:30:59.569
Ido Goldstein: Yeah, that's it. If you guys have sec- questions, let me know.

184
00:31:06.860 --> 00:31:11.870
Avraham Yoffe: Hey, the… One question, if you don't mind.

185
00:31:11.870 --> 00:31:12.540
Ido Goldstein: Yeah, of course.

186
00:31:12.540 --> 00:31:15.939
Avraham Yoffe: Plenty of… there's plenty of open, and

187
00:31:16.280 --> 00:31:21.029
Avraham Yoffe: Paid agents, available to scan,

188
00:31:21.250 --> 00:31:24.309
Avraham Yoffe: For stuff, just like you described here.

189
00:31:24.550 --> 00:31:28.930
Avraham Yoffe: Do you have an experience with these paid or non-paid services?

190
00:31:29.620 --> 00:31:33.890
Ido Goldstein: Sorry, I didn't quite understand the question. The paid service…

191
00:31:33.890 --> 00:31:39.189
Avraham Yoffe: I forgot, actually, the names of the services. There's plenty of MCP services

192
00:31:39.800 --> 00:31:47.870
Avraham Yoffe: Companies provides, provide scanning for private information, or… Or tokens keys…

193
00:31:47.870 --> 00:31:49.000
Ido Goldstein: And so on.

194
00:31:49.500 --> 00:31:54.970
Avraham Yoffe: Do you have any… your personal experience with such services, and have your own preferences?

195
00:31:55.600 --> 00:31:58.519
Ido Goldstein: Yeah, so, so, so I think,

196
00:31:58.740 --> 00:32:16.869
Ido Goldstein: I want, because I'm also a consultant, so I don't… if you want, I can discuss this privately with you, because I don't, I have clearances that, I can't recommend, but, overall, I would say that,

197
00:32:16.930 --> 00:32:24.500
Ido Goldstein: You have to have more, resp… you have to be more responsible, engineering-wise.

198
00:32:24.500 --> 00:32:40.409
Ido Goldstein: So, if, previously, if we were saying two years ago, I would say, yeah, that, SaaS, that SaaS, program, or, you know, this, this and that is, is, doing that, for me, right?

199
00:32:40.440 --> 00:32:53.700
Ido Goldstein: So today, I think, how, Spadaman says, with great power comes great responsibility. So right now, we have every open source in the palm of our hands.

200
00:32:53.700 --> 00:33:00.600
Ido Goldstein: But, we need to understand, where things are going, so…

201
00:33:00.600 --> 00:33:05.000
Ido Goldstein: There are a lot of open-source alternatives, to answer your question.

202
00:33:05.040 --> 00:33:14.419
Ido Goldstein: I think… Most of the companies that do… if I understand correctly, you spoke about Pentest, right?

203
00:33:17.960 --> 00:33:30.509
Avraham Yoffe: Plenty of things, including not only pen tests, not penetration tests, but also non-sending, not anonymized information outside of the organization.

204
00:33:30.710 --> 00:33:34.719
Avraham Yoffe: Accessing to a number of private sources internally.

205
00:33:34.980 --> 00:33:39.280
Avraham Yoffe: Actually, we don't have a very definite perimeter here.

206
00:33:39.870 --> 00:33:47.620
Avraham Yoffe: So… I agree. I just… I'm just aware there's plenty of tools trying to define such perimeter.

207
00:33:48.100 --> 00:33:52.130
Avraham Yoffe: And to work, against, leaking information.

208
00:33:52.540 --> 00:33:55.960
Avraham Yoffe: Or, exposing information the wrong way.

209
00:33:57.350 --> 00:34:07.119
Avraham Yoffe: Yeah, just for example, as a call center, you have anonymized services, I forgot the acronym, actually, calling this type of services.

210
00:34:07.270 --> 00:34:13.380
Avraham Yoffe: You can't expose everything your company holds to your… peers.

211
00:34:13.659 --> 00:34:13.979
Ido Goldstein: Right.

212
00:34:13.980 --> 00:34:20.579
Avraham Yoffe: agents. So, AI have plenty of abilities to have a host.

213
00:34:20.840 --> 00:34:22.520
Avraham Yoffe: To leak such information.

214
00:34:22.989 --> 00:34:26.539
Avraham Yoffe: I just asked you if you have an experience with the…

215
00:34:26.750 --> 00:34:31.340
Avraham Yoffe: discoveries in the existing services, not developing too much.

216
00:34:31.909 --> 00:34:38.179
Ido Goldstein: Yeah, yeah, so again, sorry that I'm answering politically correct.

217
00:34:38.329 --> 00:34:48.329
Ido Goldstein: But I have my first show of recommendation that I can't share in that platform, but I will say this.

218
00:34:48.509 --> 00:34:56.809
Ido Goldstein: I, I, I think… That, a lot of… Services, doing…

219
00:34:56.889 --> 00:35:06.939
Ido Goldstein: A lot of the same things, and it very, very much depends on what you want to safeguard in your system.

220
00:35:06.959 --> 00:35:15.119
Ido Goldstein: Each has, you know, its own niche. We can discuss it if your sensitive data sets

221
00:35:15.119 --> 00:35:36.349
Ido Goldstein: near the, the, you know, the route, if your sensitive data sits more of in the cold, of, as I said, if you have a call center, right, and everyone can reach the data, so, you, you need to practice maybe something that is more broad, and not more, more of a, like, a deep.

222
00:35:36.359 --> 00:35:40.019
Ido Goldstein: kind of defense. It really is dependent.

223
00:35:40.129 --> 00:35:56.799
Ido Goldstein: So, I'm sorry, but I don't have one solution that, you know, can rule them all, but I will say that the one who's responsible for it is the one that develops the system.

224
00:35:56.799 --> 00:36:08.339
Ido Goldstein: And it's something that is very unique right now, in nowadays, because it used to be, you know, all things regarded security, you know? There was a CISO for it, right?

225
00:36:08.339 --> 00:36:27.259
Ido Goldstein: There was a, I don't know, a security team. Right now, every developer, with the power that he can, you know, change the organization in a few days, right? You can push a few PRs containing thousands upon thousands of code lines.

226
00:36:27.369 --> 00:36:41.959
Ido Goldstein: So, with that power, that developer, that agile group that develops the system into a business goal, now need to understand security in much, much better detail.

227
00:36:41.999 --> 00:36:49.759
Ido Goldstein: So, I don't know if I, answered your question, but, it's, it's,

228
00:36:50.219 --> 00:36:53.629
Ido Goldstein: It really is a new world in that sense.

229
00:36:58.930 --> 00:36:59.610
Gabor Szabo: I love…

230
00:36:59.610 --> 00:37:00.330
Avraham Yoffe: Thank you.

231
00:37:00.430 --> 00:37:01.130
Gabor Szabo: Yeah.

232
00:37:01.310 --> 00:37:01.810
Ido Goldstein: Of course.

233
00:37:02.540 --> 00:37:06.400
Gabor Szabo: I have, I'm teaching a class, and,

234
00:37:06.560 --> 00:37:12.539
Gabor Szabo: programming using AI, and that the students are not,

235
00:37:13.090 --> 00:37:17.849
Gabor Szabo: Not programmers, they are students of researchers.

236
00:37:18.350 --> 00:37:35.659
Gabor Szabo: And they… I see how quickly they manage to generate some code that has all kind of API keys built… either embedded in the code, or saved in a configuration file, and added to Git.

237
00:37:35.800 --> 00:37:41.599
Gabor Szabo: And then pushed out to a public GitHub repository. So…

238
00:37:41.820 --> 00:37:46.549
Gabor Szabo: What you are talking about is… what you talked about is,

239
00:37:46.850 --> 00:37:52.800
Gabor Szabo: Probably relevant for all of the people… everyone who is a developer, but nowadays.

240
00:37:53.100 --> 00:37:59.730
Gabor Szabo: I think a lot more people are actually developing code, who don't…

241
00:38:00.590 --> 00:38:03.399
Gabor Szabo: Have even the… the par… the…

242
00:38:03.590 --> 00:38:15.870
Gabor Szabo: the education that we might have had, right? The official programmers, or the professional programmers might have had, or the experience. Yeah. So I wonder,

243
00:38:16.070 --> 00:38:27.880
Gabor Szabo: So, definitely, they won't be aware of tools that can scan for their… I mean, that's way above their head, or whatever, their interest, even.

244
00:38:28.070 --> 00:38:28.850
Ido Goldstein: Yeah.

245
00:38:28.860 --> 00:38:47.819
Ido Goldstein: No, of course, security is a boring field. There is a saying that everyone doesn't want to conduct the CISO until there is an exposure, and everyone then wants to discuss with the CISO or with the exposure team.

246
00:38:47.820 --> 00:38:53.529
Ido Goldstein: So, I think right now, we're getting into…

247
00:38:53.530 --> 00:39:12.029
Ido Goldstein: You can see it, online right now that, people are talking about, factories, code factories, that, just in a nutshell, it's, like, the vision that a software becomes a product.

248
00:39:12.070 --> 00:39:27.590
Ido Goldstein: Like, you can go with a spec, you have a specification of a detail that you want to set your system at, and then you will get software that does that.

249
00:39:27.850 --> 00:39:42.889
Ido Goldstein: when that day will come, I think security will be, have to build in into it in a way that produces trust, and trust

250
00:39:43.100 --> 00:39:48.460
Ido Goldstein: It's something much more, how would you say, fluent?

251
00:39:48.710 --> 00:39:54.629
Ido Goldstein: then, all the things that we discussed. So, how can we, trust

252
00:39:54.970 --> 00:39:58.550
Ido Goldstein: Code that we don't even familiar with.

253
00:39:58.710 --> 00:40:10.310
Ido Goldstein: And how, you know, to, to answer if it introduces new ways of leakage, of, you know, vulnerabilities.

254
00:40:10.400 --> 00:40:20.729
Ido Goldstein: There is no one solution for it right now. I don't think in the near year or two it will be.

255
00:40:20.800 --> 00:40:30.590
Ido Goldstein: But, we are converging. We are converging. The open source community is going out of its mind right now, really.

256
00:40:31.080 --> 00:40:43.190
Ido Goldstein: and all kinds of solutions, all kinds of ways, all kind of initiatives. In the end, I very much think that,

257
00:40:43.280 --> 00:40:59.380
Ido Goldstein: you just have to be more responsible. It's just as simple as that. You have to know, what is going in your system, and to build on a trust of how can you check yourself, how can you know that your sensitive data… you have to understand

258
00:40:59.380 --> 00:41:07.480
Ido Goldstein: what is… what you need to guard against, right? You have to understand, what, needs… needs protection.

259
00:41:07.570 --> 00:41:17.870
Ido Goldstein: So, that understanding is, going to be built up, as you prolong with, you know, business need and, you know, software.

260
00:41:20.660 --> 00:41:23.179
Ido Goldstein: That's, like, very, very general.

261
00:41:23.180 --> 00:41:23.740
Gabor Szabo: Yeah.

262
00:41:25.480 --> 00:41:29.479
Gabor Szabo: Well, any more questions from the audience?

263
00:41:31.060 --> 00:41:32.510
Gabor Szabo: Still in the video?

264
00:41:33.840 --> 00:41:44.980
Gabor Szabo: If not, then I would like to thank you for the presentation, Ido, and thanks to everyone who participated, and those people who watched the video. Now is the time to like it.

265
00:41:44.980 --> 00:41:54.870
Gabor Szabo: And follow the channel. And below the video, again, I mentioned you can find a link to notes about this event, and a link to the future events.

266
00:41:55.090 --> 00:42:10.179
Gabor Szabo: And I… and those people who are still… who are here in the meeting, you can stay on, and after I stop the video recording, we can continue discussing things that we didn't want to, live, on the record.

267
00:42:10.390 --> 00:42:15.710
Gabor Szabo: So, thank you again, and see you at one of our next sessions.

268
00:42:16.100 --> 00:42:16.970
Gabor Szabo: Bye-bye.

269
00:42:17.600 --> 00:42:18.650
Ido Goldstein: Bye-bye, guys.

