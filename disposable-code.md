# Disposable Code

Code is disposable in the context of serving a business model.
A for-profit company must have a business model.
Restaurants get paid for serving food, dining experience, etc.
Retail stores buy for less, sell for more.
Car manufacturers make and sell cars.
Even air, which is abundant and free, can be part of a business model,
as long as people are willing to pay what you sell.

Code is written with some implicit goals in mind.
It must be readable, maintaniable, easy to extend.
With such goals in mind, code was carefully designed, implemented, peer-reviewed, tested, and released.
Remember, all those phases are iterative.
At any later stage, we might need to jump to previous phases to redo it again.

Code is undeniably expensive.

Rarely do developers consider when a piece of code should be deprecated, deactivated, and decommissioned. It was written as if it has perpetual value.
But it hasn't.

Some code survived, but majority of them disappeared along time.
This is especially significant for a company.
Rare is the case that a piece of code written 7 years ago still plays a role in today's business.
If Microsoft declares that they will stop Windows development, a few years later, it would be hard to see any significant revenue from Windows sales.
If Apple declares no more macOS updates in the future, not only their software business will suffer, their hardware sales will also plummet.

Writing code as if it runs forever costs more resources.
Since we expect it to run for a long period time.
Because of the expection of code to serve us for long, an increased amount of efforts are spent on it.
This self reinforcement makes code even more expensive.
It takes longer time to write, costs more to test and maintain.
But there must be a point where this spiral circle breaks.

When the inevitable decommissioning happens, it will be hard to handle since it was written without such events in mind.

How well a piece of code should be written?
How much test is enough?
And, finally, when would be the time to consider to end the support of it?

Those questions aren't answered well, even for well established companies.

We had software lifecycle management for packaged software. Packaged softwares have distinctive lifetime. As the industry shifts to cloud computing, it is much blurry now. You probably heard anecdotes about a hidden linux box sitting under someone desk running some vital services that a lot of other services are relying on, and being noticed only when it was accidentally shutdown or taken away.

Code, however, is mostly hidden behind the scene, and never the focus of lifecycle management.

Often than not, code was written as well as a group of people collectively can.
Unit tests and integration tests are kind of hit-and-miss.
Manual tests are either done by the developer who wrote it, or by others, if time permits.

Once a piece of code is released to the wild, it will be there running, 
until it breaks, at which time, an issue will be reported,
and propagated through multiple tracking systems,
and eventually reached developers who might need to read a lot to figure out what was going on.
What made situations worst is that nobody planned for this work.
Even if it was indeed planned, it is hard to predict how much work is needed.
How hard it is to solve.
As time passing by, same issue becomes hard to fix because of the churn,
the loss of documentation, the intervined dependencies built upon in those years.
It is not visible until a quitting spree caused by burnouts.

In conclusion, before writing a piece of code, it is better to first set the scope based on its business value, then, in turn, plan related efforts accordingly so that the team won't waste time on it.
Quality is relative. You need to decide how good is good enough.
For frequent-running code, performance is important.
For occasional-running code with long lifecycle, readability and maintainability trump performance.
A piece of code has its limit.
If you expect a piece of code to constantly serve a business goal, plan ahead to allocate resource accordingly. Otherwise, this debt will be paid later at the least convenient time possible, and worst, with some severe consequences, like data breach.
