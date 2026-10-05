# Read the Book of Why

# read The Book of Why

Denali Lumma denali@doubling.io via googlegroups.com 
	
AttachmentsFri, Sep 25, 5:49 PM (9 days ago)
	
	
to dora-community
Encouraged by Storey's recent DORA presentation covering Cognitive and Intent Debt https://queue.acm.org/doi/10.1145/3807966

I became interested in "The Book of Why," because I thought it might shed light on how we could move toward more structured "thinking" in AI, and perhaps address the growing debt we now face:

https://www.amazon.com/Book-Why-Science-Cause-Effect/dp/1541608984

I discovered that the paper below captures the heart of the book:

https://projecteuclid.org/journals/statistics-surveys/volume-3/issue-none/Causal-inference-in-statistics-An-overview/10.1214/09-SS057.full

My understanding, in short:

Experiments are the clearest way to learn cause and effect, but they're expensive, slow, and sometimes impossible. The alternative is to use the data you already have. When you notice a trend, draw what you believe drives it as a diagram, write the question in do-notation, and check whether your data can answer it. If it can, compare within the things that drive both sides so they cancel out, and calculate the answer. If it can't, pick an alternative. The answer is only as good as the diagram, so your assumptions are written down where others can challenge them. This works even when you can run an experiment, because it tells you whether you need one.

It doesn't give AI a formal thinking structure, but it does offer a precise format for one kind of intent: the causal beliefs behind decisions ("we're doing X because we believe it will cause Y"), written down in a form that humans and AI agents can check later.

My full notes are attached.

-Denali
