# Talk transcript · Bridging Classical and Data-Driven Transport Modelling

**Topic: methods for traffic-flow prediction modelling** · 2026-07-04 · Dr Yue Li · University of Cambridge

> Transcribed from a recording of the talk and **organised in slide order**; only the formal presentation is kept — the pre-talk chat, the Q&A, and the post-talk discussion have been removed. The spoken language has been cleaned up and lightly edited. Each heading gives the slide number(s) (1–62) and title. *(Translated from the Chinese delivery.)*

---

## Slide 1 · Opening (Title)

Hello everyone, I'm Yue Li, from the Department of Architecture at the University of Cambridge. The department has a few strands: transport modelling (mine), urban planning, and a rather different one on architectural history. So when I say I'm in "architecture", people sometimes assume I draw buildings. What I'd like to share today is how to apply computational modelling to transport.

## Slide 3 · Why model traffic at all?

Let me state the problem first: we want a model that tells us roughly how many vehicles are on every road, ideally at good spatial and temporal resolution. That's the goal.

There are many downstream uses. For instance, studying the impact of climate change on urban systems — there's a lot of resilience research now: extreme weather brings extreme rainfall and flooding, which submerges roads; once roads are disrupted, how do people adjust their travel (shopping, commuting), and how badly are they affected? Such scenario analysis needs traffic estimates. Or public health — my master's was in environmental health at the Harvard School of Public Health, which involves exposure assessment: traffic directly affects the air pollution and noise around homes, and hence health. Our group has recently found direct evidence linking traffic to mental health, and even to dementia.

## Slide 5 · What a good answer must have

So "being accurate" is only the most basic requirement. Concretely, we care about a few dimensions: first, giving sensible estimates on road segments with no observations; second, supporting long-term planning — factoring in future socio-economic and population change to produce reasonable future-scenario estimates; third, transferability — after building a model where data is rich, transferring it to data-sparse regions (e.g. in Africa) to help them build their own transport models, which would be very valuable. In short, four properties: the model must be **accurate**, have **spatial transferability** and **temporal transferability**, and be **interpretable**.

## Slides 6–7 · The four-step travel demand model

On transport modelling, there's a body of classic theory going back to the 1970s–80s that is almost the "bible" of this field: the travel demand four-step model. It breaks "how trips arise and end up as flow on each road" into four steps: trip generation, trip distribution, modal split, and traffic assignment. Let me go through them.

## Slides 8–9 · Step 1 — Trip generation / attraction

Step 1 is trip generation, together with its counterpart trip attraction. You first divide the map into many zones, and for each zone estimate how much travel demand it produces and how much travel it attracts.

How to model it, and with what features? The simplest is linear regression: trips produced relate to a zone's population, car ownership and socio-economic status; trips attracted relate to the employment opportunities in the zone. Another approach splits by population segment — e.g. crossing the regression variables into cells like "aged 16–64 / male / a given socio-economic status", each cell having a fixed trip-production rate. Both are essentially the same: given a zone's features, output a trip rate (say, roughly how many trips in the morning peak hour).

## Slides 10–11 · Step 2 — Trip distribution and the gravity model

Step 1 gives each zone's production and attraction — i.e. the row and column totals of an OD matrix. Step 2, trip distribution, fills in "how many trips go from each zone to every other zone" given those row and column totals.

The classic tool is the gravity model, much like Newtonian gravity: two zones that are closer and larger (in production/attraction "mass") exchange more trips. Each cell is governed by the cost between i and j (time or money); larger cost means weaker attraction, captured by an exponential decay — the deterrence.

The catch: cells computed this way won't, when summed by row and column, generally match the totals from step 1 — there are too many constraints. So we use the Furness iteration (bi-proportional fitting): scale each row to hit its target total; after that the columns no longer match, so scale each column; that unbalances the rows again, so you iterate back and forth until it converges.

## Slides 12–13 · Step 3 — Modal split and generalised cost

The first two steps only concern how many trips are produced and attracted, not the mode. Modal split considers walking, cycling, car, rail, bus, etc. — modes that substitute for one another.

Modelling this also uses a generalised cost, converting the factors into a common currency of time or money, then a softmax choice: each option is weighted by exp(−cost), the cheaper the larger the share; after normalisation you get the probability of choosing each mode — a discrete choice model. The generalised cost can include in-vehicle time, waiting time for public transport, congestion charge, motorway tolls, parking fees, and so on. Nicely, if you put both time and money in, you can back out the coefficient at which a person trades time for money.

## Slides 14–15 · Step 4 — Assignment and equilibrium (UE vs SUE)

After the first three steps, each mode has its own mode-specific OD matrix. But an OD matrix is abstract — it only says "these people want to go from here to there"; it hasn't landed on the real road segments. Step 4 lets these trips choose routes in physical space.

You must set behavioural parameters: faced with several routes, why choose this one over another? Two mainstream options. The first, most common and simplest: assume everyone is rational with perfect information and picks the route with the lowest generalised cost. The second, stochastic user equilibrium (SUE), used less often: assume choices aren't fully rational — people have preferences and can't be captured perfectly by lowest cost, and this deviation is called noise; route choice becomes a discrete choice too, distributing probability by route cost, with a controllable sensitivity to cost differences.

The key point: everyone takes the route they think is fastest (say, all onto the motorway), but the motorway isn't infinitely fast; in the morning peak, everyone piling onto the same road causes congestion, which raises that road's travel time — i.e. its cost — which in turn changes people's choices. That's a loop. UE and SUE seek the equilibrium of this loop, and it can be proven mathematically that such a convergence point exists and is findable.

## Slides 16 & 36 · Strengths and limitations of the four-step model

The strength of the four-step model is that every step is human-understandable — each abstracts and models human behaviour. Almost every large commercial model today is built on the four-step backbone, just with variations. For example, one very strong current model, from a group led by a professor in Berlin, treats each person as an agent — the simulation unit is the individual: each person decides when to leave, what to do, which mode to take; the whole population becomes NPCs choosing routes, and parameters are tuned to reproduce a region-wide simulation.

More importantly, the four-step model is sequential and interpretable, so policy makers trust it — they're accountable for the results and must understand how the model works, with nothing inexplicable inside.

But the limitations are clear: the steps are treated as independent; and almost every city must calibrate its own full model. The behavioural parameters should in principle transfer (people's sense of time and money is broadly similar), but in practice this isn't achieved — every region calibrates from scratch, which takes months and a lot of money; and it's hard to use very fine zones, or the computation explodes.

## Slides 18–20 · Machine learning: regression and nonlinearity

The other direction is machine learning; start with regression. Regression is for tabular data: a Y (here, traffic) and some X, using X to predict Y. Being linear, it's solved by OLS minimising MSE, and it's very interpretable. You can extend it to break the linearity assumption — adding polynomial terms, splines, interaction terms to make it more complex — but the more you add, the less interpretable it gets; it's a matter of degree.

## Slides 21–23 · Tree models: decision trees, random forest, boosting

Another big family is tree-based models. At the bottom is the decision tree: at each node you set a threshold on some feature and split in two, choosing the split that minimises the "entropy" of the two resulting groups, and keep splitting.

On top of trees is the random forest: train many "weak" trees — each uses only some features (about √n of the n features, chosen at random) and bootstraps the samples (sampling with replacement, mimicking repeated sampling of the real world). A single tree is weak, but train hundreds, let each vote, and average — that's the random forest.

There's also a stronger family (boosting), which in most settings does beat the random forest: train one tree, and since there's a residual (imperfect fit), train the next tree on the previous round's residual; keep fitting the residuals with more trees, then sum every tree's prediction. These methods are very accurate on tabular data — essentially state of the art.

That said, what a random forest actually learns is beyond direct human comprehension. A common way to explain it is the SHAP value — a post-hoc analysis: after training, relate the original features to the predictions, effectively reducing the model to a monotonic, human-readable pattern while trying to keep the accuracy.

## Slides 24–25 · Neural networks: the forward and backward pass

The neural network is a whole other approach; its most basic element is the MLP (multi-layer perceptron). The first layer is all inputs x; each hidden node is "each input times its weight plus a bias", e.g. the first hidden node = w₁x₁ + w₂x₂ + w₃x₃ + b; four hidden nodes mean four sets of weights and biases; the output Y is likewise a weighted sum of the hidden nodes plus a bias. That's the simplest neural network.

Its parameters start random, so without training the output is a guess. How do we adjust W and b to make Y accurate? With the deep-learning approach: take the residual between Y and the truth, take partial derivatives with respect to all preceding parameters, see how a small change to each parameter would make Y better, then descend along the gradient. You go step by step because you can't solve for every parameter's direction at once — you can only find one parameter's direction while holding the others fixed; once you change it, the best directions of the others change too, so at each step you re-evaluate — like descending from a peak to a valley: you know the rough direction, but each step follows the locally steepest descent. The machinery is the chain rule: each layer's gradient can be computed and the gradient signal passed backward layer by layer, however deep the network, letting it fit very complex nonlinear functions.

## Slide 26 · From tables to structure

All of the above is modelling on tabular data. But transport modelling isn't that easy — the hard part is **how to construct that table**: given a road, which information do you collect as its X? A naive approach draws a buffer (box) around the road and collects the population, jobs, etc. inside as features. This is crude: a road might be a motorway exit or a quiet corner in a busy district — the box may contain lots of people who have nothing to do with the road. The box doesn't capture "how real travel demand actually flows through this road", so a purely tabular approach is very limited. This motivates neural-network variants tailored to different data structures.

## Slide 27 · Convolutional neural networks (CNN)

If the data is 2-D (like a photo), everyone knows to use a CNN. A CNN's strength is recognising 2-D features in an image: the deeper the layer, the larger the thing it extracts — first a cat's ear-tip, then the cat's face. It typically uses a 3×3 matrix as a learnable weight, sliding this small window over the image, multiplying with pixels and summing into the next layer — what it learns is what values that 3×3 window should take to extract useful features.

Why better than tabular? Take handwritten digits 1–9 photographed as 30×30 black-and-white pixels: you could flatten that to a 1×900 vector and feed a tabular model — it would have some accuracy, but not enough: much of the information is unnecessary, the model is heavy and inefficient, and once flattened you lose the pixels' 2-D spatial relationships (a slightly slanted digit is no longer recognised). A CNN preserves exactly those spatial relationships. And a map, like a photo, is 2-D data, so CNNs apply here too.

## Slide 28 · Graph attention networks (GAT)

Besides CNNs for 2-D grids, we can use graph neural networks to learn these relationships. A graph neural network does something similar to a CNN but breaks the grid constraint: a graph can be any topology — not a 3×3 grid; a centre node can connect to three neighbours, or five. And a transport network is inherently a topology, which suits graph neural networks.

There are two main types: GCN (graph convolutional network) and GAT (graph attention network). GAT essentially moves the attention mechanism from the Transformer — the core component of large models — onto the graph. Each node has a feature vector, and each edge has one too; one operation (one layer) aggregates all neighbours' information into the centre node — each neighbour's feature is multiplied by a matrix, then combined with a weight to update the centre. That weight is set by attention, which is itself determined by the features of the neighbour node, the centre node, and the edge between them; the weights sum to 1. The graph thereby learns how to aggregate neighbours' information into the centre node sensibly.

> (A personal note: learning deep learning is really about developing a feel for "how information flows" — what you're building is a bridge for information to flow along.)

## Slides 29–30 · Recurrent networks and Transformers (RNN / Transformer)

Another type is the RNN, for sequences. Its input–output structure is the same as a Transformer's: feed in a sequence, produce a sequence (the Transformer's original version actually came from RNNs). What is it for? If traffic is a time series, you can use the past 12 time stamps to predict the next 12 — that's the typical application. The Transformer I've already mentioned, so I'll skip it here.

## Slide 32 · The short-term model's "sandwich" structure

Once you know these components, you can "stack building blocks". The basic rule is: the input and output dimensions must match, information must flow effectively, and you end with an output. And a "first-principles" belief I hold: if a person given these inputs could roughly guess whether traffic is high or low, then in principle the machine can learn it too; conversely, feeding a pile of meaningless features and asking the machine to learn an impossible task won't work.

Here is a very classic short-term traffic-prediction architecture: an RNN first encodes the historical information; a graph attention network then aggregates the surrounding spatial information and updates each node; then the temporal information is updated again, followed by another RNN — a "time–space–time" sandwich. After training you add a decoder, outputting one step and feeding it back, to produce the next 12. What it does is exactly "past 12 in, future 12 out".

## Slide 33 · The short-term model is "one step behind"

This model has a big problem, and the field competes very fiercely on it. The problem: to predict the traffic at some moment, must you really know the immediately preceding 12 time stamps? A time stamp is usually 5 or 15 minutes. If I want to predict this time next year, must I first know the traffic two hours before that moment next year? That's a bit absurd. It's fundamentally a short-term model — training chops the data into "12-in / 12-out" windows — so it can only do short-term, not long-term.

Plot its predictions and you see: each predicted point is extrapolated from the previous 12 real points, and it's almost always "one step behind" — congestion is about to start next step, speed and flow about to drop, yet it still thinks everything is fine. If even predicting congestion is this hard, its usefulness is limited.

But this kind of model is great for publishing papers: it's accurate, extrapolation isn't far off, MAPE is often under 5%, the metrics look good. So a lot of people in computer science keep grinding on it — every 0.0-something drop in R² or MAPE gets claimed as beating the current best. The novelty is limited: often it's just a new stack of blocks that "somehow works", the metric improves, and you can't say it isn't better. In fact, having read their code, some haven't even done basic data preprocessing — missing values are just dumped straight in.

## Slides 34–36 · Back to the gaps: the two paradigms compared

Recall the goals: accurate, spatially and temporally transferable, and interpretable. The traditional four-step model's problem is being over-simplified, but it can predict 1, 5, or 20 years ahead — just re-calibrate. Today's deep-learning models are all short-term: they don't take the four-step model's features as input, so they can't do long-term planning, at best short-term monitoring. Each paradigm fails in its own way; neither is accurate, transferable, and interpretable at once.

## Slides 37–39 · Data

We collected data in the UK: the road network, high-quality socio-economic features, and traffic data on the major trunk roads at roughly one value every 15 minutes.

## Slides 40–44 · Mukara: the first model

The first model just "stacks blocks": place a node at every junction of the road network, use a CNN around each node to collect the surrounding population, jobs and other spatial information, and aggregate that via the CNN into the initial node vector the GAT needs; the GAT is built on the trunk-road structure (the network is hand-constructed), and lets information propagate over the graph; after propagation, relate the final node vectors to Y and train.

## Slide 45 · Mukara's results

The result isn't very accurate — no match for short-term models — but at least there's a signal.

## Slide 46 · What Mukara cannot do

Reflecting on its problem: it's too much "block-stacking". CNN plus GAT does work, but it dumps all inputs in without embedding the essence of the four-step model. I think the ideal deep-learning model should also draw on theory in how information flows. Two lessons: first, feature engineering matters enormously — good feature engineering beats a very complex model; second, architecture engineering — if you fold the theory's information-flow structure into the network, you save a lot of effort. In short, let the model learn only what it genuinely should, and don't hand it tasks a human can do accurately.

## Slides 48–52 · DeepDemand: embedding theory into the network

Hence a second version. Given a road, first work out "where can get onto this road, and where does this road lead" — using a competitive shortest-path algorithm to cleanly identify the road's candidate origin (O) and destination (D) regions; then take the O and D zones, pair them into OD pairs, and keep only those OD pairs that genuinely use this road (some O–D pairs simply don't need to pass through it; only pairs whose shortest path must cross this road count).

## Slides 53–55 · DeepDemand's results and comparison

This purpose-built model does much better — currently around R² ≈ 0.7. More importantly, it transfers across regions: in spatial cross-validation, train on 8 of 9 regions and test whether it works on the completely unseen 9th — the result is quite good. Publishing needs baselines, so I compared against linear models and tabular tree-based models (making those methods work on tabular data too).

## Slides 56–58 · Interpretability: what the model learned

Since the model can essentially be read as a mimic of the four-step structure, I looked further at whether it's interpretable. For instance: the "travel time vs travel probability" function it learned on its own (with no supervision); the relationship between each O/D zone's generation/attraction strength and its raw features; and hotspots the model itself picked out from road traffic — which turn out to be airports, even though I never told it those were airports.

## Slides 60–61 · What comes next

Next I'd like to take a "white-box" approach: starting from the most primitive behavioural parameters of the four-step model, and using the deep-learning training machinery, to infer those behavioural parameters back out. That's roughly the work ahead.

## Slide 62 · Closing

I'll stop here — let's see what questions you have.
