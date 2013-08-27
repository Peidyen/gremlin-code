/*
 * Example of a label propagation community detection algorithm implemented in Gremlin.
 *
 * The graph we're using here is a track-[:hasTag]->tag kind of graph, and we're detecting
 * communities based on the graph that connects tracks to tracks (via tags with a weight
 * greater or equal to 80). In this example, we do not detect convergence to avoid adding
 * overhead since our test graph was quite large.
 *
 * Note that, for large-scale graphs, this algorithm takes a long time to complete if
 * there aren't enough resources available. While I haven't tested it, I believe a highly
 * available Neo4j cluster would be beneficial.
 */

/*
 * Set all node labels to their own ID.
 *
 * (Yes, you could use the ID instead of running this first iteration,
 * but if you want to run the algorithm again at a later time, you won't
 * know whether to use the ID or the old label to propagate.)
*/
println "Initializing node label using node ID..."
g.idx("tracks")[[trackId:"%query%*"]].sideEffect{
	it.setProperty("propagationLabel", it.getId()) };

/*
 * Do one iteration of label propagation and decide whether to continue or not.
 *
 * Here we use a boolean to control whether every node has the most frequent label
 * over all its neighbors, but we also use a limit for the number of iterations to
 * avoid being stuck with ties.
 */
rand = new Random();
limit = 10; // According to the author of the algorithm, 95% of the nodes are well classified after iteration 5.
converged = true; // Should be used to stop the loop before iterations limit if it converges.
(1..limit).each{ i ->
	println "Iteration " + i + " started at " + new Date();
	g.idx("tracks")[[trackId:"%query%*"]].sideEffect{
			println it
			neighborLabelCount = [:];
			it.outE("hasTag").has("weight", T.gte, 80.0f).inV
				.inE("hasTag").has("weight", T.gte, 80.0f).outV
				.except([it]).propagationLabel
				.groupCount(neighborLabelCount).iterate();
			max = neighborLabelCount.values().max();
			topLabels = neighborLabelCount.grep{it.value == max};
			newLabel = topLabels.size() == 0 ? it.getProperty("propagationLabel") :
				topLabels[rand.nextInt(topLabels.size())].key;
			it.setProperty("propagationLabel", newLabel);
		}.iterate();
}
