graph d3.js Implementation

	if Meteor.isClient
		"abc"
	# D3 js Implementation - not working
	### 		
		@updateGraph = (graph, connections, contacts) ->

			y = $(".graph").height() / 2
			x = $(".graph").width() / 2
			nodes = graph[0]?.contacts

			if not nodes? 
				console.log "graph is empty"
				console.log graph
				return

			links = []

			svg = d3.select("svg")
			if not svg[0][0] then svg = d3.select(".graph").append("svg").attr("width", x * 2).attr("height", y * 2)

			force = d3.layout.force()
				.gravity(-.5)
				.distance(40)
				.charge(-200)
				.size([x * 2, y * 2])
				.on("tick", tick)

			link = svg.selectAll(".link")
			node = svg.selectAll(".node")

			drag = force.drag().on("dragstart", dragstart)
			dblclick = (d) -> d3.select(this).classed("fixed", d.fixed = false)
			dragstart = (d) -> d3.select(this).classed("fixed", d.fixed = true)
			tick = () ->
				link.attr("x1", (d) -> return d.source.x )
					.attr("y1", (d) -> return d.source.y )
					.attr("x2", (d) -> return d.target.x )
					.attr("y2", (d) -> return d.target.y )
				node.attr("transform", (d) -> return "translate(" + d.x + "," + d.y + ")" )

traveling salesman

			links = connections.filter((n) -> nodes.indexOf(n.user) isnt -1).map((n) -> ({ source: nodes.indexOf(n.user), target: nodes.indexOf(target) } for target in (n.contacts.map((c) -> c.id )?.intersect(nodes) ? []))).flatten()

			spiralTile = (tileNum) ->
				intRoot = Math.floor(Math.sqrt(tileNum))
				
				_x = Math.round(intRoot/2)*Math.pow(-1,intRoot+1)+(Math.pow(-1,intRoot+1)*(((intRoot*(intRoot+1))-tileNum)-Math.abs((intRoot*(intRoot+1))-tileNum))/2)
				_y = Math.round(intRoot/2)*Math.pow(-1,intRoot)+(Math.pow(-1,intRoot+1)*(((intRoot*(intRoot+1))-tileNum)+Math.abs((intRoot*(intRoot+1))-tileNum))/2)

				if (_x is 0 and _y isnt 0) or (_x isnt 0 and _y is 0) then m = Math.pow(Math.sin(Math.sqrt(Math.pow(_x, 2) + Math.pow(_y, 2)) + Math.PI / 2), 2) + 1
				else m = 1

				x: _x
				y: _y
				m: m

			nodes = nodes.map (node, i) ->
				tile = spiralTile(i)
				
				id: node
				weight: 1
				x: tile.x * 100 * tile.m + x
				y: tile.y * 100 * tile.m + y
				avatar: contacts.find((n) -> n.id is node).avatar

			console.log nodes
			console.log links

			link = svg.selectAll(".link")
				.data(links)
				.enter().append("line")
				.attr("class", "link")

			node = svg.selectAll(".node")
				.data(nodes).enter()
				.append("g")
				.attr("class", "node")
				.on("dblclick", dblclick)
				.call(force.drag)
				.append("image")
				.attr("xlink:href", (d) -> d.avatar	)
				.attr("x", (d, i) -> d.x)
				.attr("y", (d, i) -> d.y)
				.attr("width", 64)
				.attr("height", 64)
			
			force
				.nodes(nodes)
				.links(links)
				.start()
	###