---
layout: post
title: Rule of Simplicity
date: 2026-03-28
image: /blog/images/05-rule-of-simplicity.png
author: Jason M Penniman
excerpt: "Rule of Simplicity: Design for simplicity; add complexity only where you must."
tags:
- software engineering
- unix philosophy
---

> Rule of Simplicity: Design for simplicity; add complexity only where you must.

# Introduction

The Rule of Simplicity is one of the foundational guidelines in the Unix philosophy, famously articulated by Doug McIlroy and later summarized by Eric S. Raymond. It states that software should be designed to be as simple as possible, making it easier to understand, maintain, and extend. Simplicity means doing only what's necessary, and doing it well. (See the [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy) and Eric S. Raymond's _The Art of Unix Programming_.)

# Why Simplicity Matters

Simplicity in our software design brings many benefits:

- **Maintainability:** Simple code is easier to read, debug, and modify.
- **Reliability:** Fewer moving parts mean fewer opportunities for bugs.
- **Composability:** Simple components can be combined in powerful ways.
- **Accessibility:** New contributors can understand and work with the code-base more quickly.

# Applying Simplicity

Modern software development can benefit from the Rule of Simplicity:

- **Write small, focused functions and modules.**
- **Avoid unnecessary features or options.**
- **Prefer clear, readable code over clever tricks.** (See the [*Rule of Clarity*](/rule-of-clarity))
- **Document intent and usage simply.**

How to apply (quick checklist):

- Keep functions and methods small and focused; prefer composition over deep inheritance.
- Enforce single responsibility in code reviews and pull requests.
- Remove or postpone features that don't have clear user value.
- Minimize and periodically audit dependencies.
- Write clear tests and small examples that demonstrate intended behavior.

When in doubt, ask: "Is there a simpler way to achieve this?"

# Real-world C# Refactor Example

Below is a short before/after that shows how applying the Rule of Simplicity reduces complexity and improves testability.

Before (monolithic method doing several responsibilities):

```csharp
public class UserService
{
	public void RegisterAndNotify(User user)
	{
		// validate
		if (string.IsNullOrWhiteSpace(user.Email))
			throw new ArgumentException("email");

		// persist (inline DB code)
		using(var conn = new SqlConnection("..."))
		{
			// complex insert logic
		}

		// logging
		Console.WriteLine($"Registered {user.Id}");

		// email
		var smtp = new SmtpClient();
		smtp.Send(user.Email, "Welcome", "...");
	}
}
```

Problems: mixes validation, persistence, logging, and emailing — hard to test and change.

After (single responsibility + DI):

```csharp
public interface IUserRepository { void Add(User user); }
public interface IEmailSender { void Send(string to, string subject, string body); }

public class UserService
{
	private readonly IUserRepository _repo;
	private readonly IEmailSender _emails;
	public UserService(IUserRepository repo, IEmailSender emails)
	{
		_repo = repo; _emails = emails;
	}

	public void Register(User user)
	{
		Validate(user);
		_repo.Add(user);
		_emails.Send(user.Email, "Welcome", "...");
	}

	private void Validate(User user)
	{
		if (string.IsNullOrWhiteSpace(user.Email))
			throw new ArgumentException("email");
	}
}
```

Benefits: each piece is testable and replaceable (mock `IUserRepository`/`IEmailSender`), code paths are smaller, intent is clearer, and adding features (e.g., transactional behavior or retry logic) can be done in a focused place.

# Common Pitfalls

Some common traps that violate the Rule of Simplicity include:

- **Overengineering:** Adding features or abstractions that aren't needed. Most abstractions and layers of indirection are unnecessary.
- **Feature creep:** Letting a component grow beyond its original purpose.
- **Obscure code:** Writing code that's clever but hard to understand.

To avoid these, regularly review your code and designs, and seek feedback from others.

# When complexity is necessary

There are valid reasons to accept added complexity: significant performance constraints, security or regulatory requirements, or platform limitations may demand more sophisticated designs. When complexity is required, document the rationale, add tests, and restrict the scope, so the complexity doesn't spread across the codebase.

Compress that complexity--reduce/remove the friction it causes--whenever possible.

# The Age of Agentic Coding

As AI-powered tools and coding agents become more prevalent, the Rule of Simplicity is more important than ever. Agentic coding—where autonomous or semi-autonomous AI agents (e.g. Claude Code, GitHub Copilot) assist in writing, refactoring, or maintaining code—relies on clear, simple code to be effective. These tools work best when the codebase is well-factored, documented, and covered by tests.

**Why does simplicity matter for agentic coding?**

- **Interpretability:** Agents can better understand and modify code that is straightforward and well-structured.
- **Automation:** Simple code is easier to analyze, test, and transform automatically.
- **Collaboration:** Human developers and AI agents can work together more efficiently when the codebase is not burdened by unnecessary complexity.

By leveraging the Rule of Simplicity, teams can maximize the benefits of agentic coding, reduce friction, and ensure that both humans and machines can contribute productively to software projects.

# K.I.S.S. Principle

Perhaps one of the most important engineering principles is: Keep It Simple and Stupid (KISS). Clarence Johnson is credited
with coining the term, and his message resonates with the Rule of Simplicity. When his team at Lockheed was tasked with
designing a plane for the US Military, he said (paraphrased) "This plane must be able to be maintained and repaired, in
the field, by an average mechanic with only a basic set of tools." It has been a key engineering principle ever since.

So what does that mean for Software Engineering? It means our software and systems should be able to be maintained and supported by
the average developer/tester/operations/support with a basic set of tools. Keeping our code, and our solutions, simple
and clear achieves this goal.

# Conclusion

The Rule of Simplicity is timeless advice for software engineers. By striving for simplicity, we create tools and systems that are robust, maintainable, and a pleasure to use. In the age of AI-assisted coding agents, this is more important than ever. Let simplicity guide your next project; refactor towards simplicity in your existing projects.

Cheers!
