Routing

	if Meteor.isClient

		Router.configure {
			layoutTemplate: "layout"
		}
		Router.onBeforeAction(AccountsTemplates.ensureSignedIn, {
			except: ['security', 'disallowed']
		})
		Router.map () ->
			@route("security", {
				path: '/'
				template: "pw"
				onBeforeAction: () -> 
					if not Meteor.user()? then Meteor.call "deprecateXing"
					else Router.go "home"
			})
			@route("disallowed", {
				path: '/disallowed'
				template: "disallowed"
			})
			@route("home", {
				path: '/home'
				template: "anonymousSwitch"
				yieldTemplates:
					dummys: { to: "profiles" }
					comments: { to: "content" }
				onBeforeAction: () -> Meteor.call "prepareXing"
				waitOn: () -> [Meteor.subscribe("dummys", {}), Meteor.subscribe("comments", {}), Meteor.subscribe("journalists", {})]
				data: () -> 
					contacts: Dummys.find {}
					comments: Comments.find {}
					journalists: Journalists.find {}
				onData: () ->
					data = @data()
					Session.set "users", data.contacts.fetch()
					Session.set "comments", data.comments.fetch()
					Session.set "journalists", data.journalists.fetch()

			})
			@route("cockpit", {
				path: '/cockpit'
				template: "personalSwitch"
				yieldTemplates:
					contacts: { to: "profiles" }
					graph: { to: "content" }
				waitOn: [Meteor.subscribe("contacts", user: Meteor.userId()), Meteor.subscribe("connections"), Meteor.subscribe("graph", user: Meteor.userId())]	
				onRun: () ->
					foo = () -> 
						Meteor.call "get", (err, result) -> 
							if result?.success is false then Router.go "home"
							else if result?.success is true then Session.set "loading", false				
							else if err? then foo()
					Session.set "loading", true

					@render()
					foo()
				data: () -> 
					contacts: Contacts.find {}
					connections: Connections.find {}
					graph: Graph.find {}
				onData: () -> 
					data = @data()
					Session.set "contacts", data.contacts.fetch()
					Session.set "connections", data.connections.fetch()
					Session.set "graph", data.graph.fetch()


					h = $(".graph").height()
					w = $(".graph").width()
					
					nodes = window.nodes ? []
					edges = window.edges ? []

					update = () ->
						_nodes = Session.get("graph")[0]?.contacts
						_connections = Session.get "connections"
						_contacts = Session.get "contacts"

traveling salesman

						_edges = _connections.filter((n) -> _nodes.indexOf(n.user) isnt -1).map((n, i) -> ({ id: n.user + "--" + target, weight: Math.random() * 7, source: _nodes.indexOf(n.user), target: _nodes.indexOf(target) } for target in (n.contacts.map((c) -> c.id )?.intersect(_nodes) ? []))).flatten()

						spiralTile = (tileNum) ->
							intRoot = Math.floor(Math.sqrt(tileNum))
							
							_x = Math.round(intRoot/2)*Math.pow(-1,intRoot+1)+(Math.pow(-1,intRoot+1)*(((intRoot*(intRoot+1))-tileNum)-Math.abs((intRoot*(intRoot+1))-tileNum))/2)
							_y = Math.round(intRoot/2)*Math.pow(-1,intRoot)+(Math.pow(-1,intRoot+1)*(((intRoot*(intRoot+1))-tileNum)+Math.abs((intRoot*(intRoot+1))-tileNum))/2)

							if (_x is 0 and _y isnt 0) or (_x isnt 0 and _y is 0) then m = Math.pow(Math.sin(Math.sqrt(Math.pow(_x, 2) + Math.pow(_y, 2)) + Math.PI / 2), 2) + 1
							else m = 1

							x: _x
							y: _y
							m: m

						_nodes = _nodes.map (node, i) ->
							tile = spiralTile(i)

							id: node
							x: tile.x * 50 + w / 2
							y: tile.y * 50 + h / 2
							size: Math.round(Math.random() * 32)
							color: 'teal'
							type: "circle"
							image: 
								url: _contacts.find((n) -> n.id is node).avatar
								clip: 0.7


						dragstart = (d) -> d3.select(this).classed("fixed", d.fixed = true)
						dblclick = (d) -> d3.select(this).classed("fixed", d.fixed = false)
						create = () ->
							force = window.force
							d3.select(".graph").select?("svg").remove?()
							svg = d3.select(".graph").append("svg").attr("width", $(".graph").width()).attr("height", $(".graph").height())
							link = svg.selectAll(".link")
							node = svg.selectAll(".node")
							pattern = svg.selectAll(".pattern")

							force: force
							svg: svg
							link: link
							node: node
							pattern: pattern
						tick = () ->
							o = xinnectGraph
							o.link.attr("x1", (d) -> d.source.x )
								.attr("y1", (d) -> d.source.y )
								.attr("x2", (d) -> d.target.x )
								.attr("y2", (d) -> d.target.y )
							o.node.attr("transform", (d) -> "translate(" + d.x + "," + d.y + ")" )
							#o.node.attr("cx", (d) -> d.x - 24)
							#o.node.attr("cy", (d) -> d.y - 24)
						init = (o = xinnectGraph) ->
							o.force.nodes(o.nodes).links(o.links).start()
							o.link = o.link.data(o.links).enter().append("line").attr("class", "link")
							#o.pattern = o.pattern.data(o.nodes).enter().append("pattern").attr("class", "pattern").attr("x", 0).attr("y", 0).attr("width", 80).att("height", 80).attr("id", (d, i) -> "image" + i).append("image").attr("xlink:href", (d, i) -> o.nodes[i].image.url).attr("x", -8).attr("y", 0).attr("width", 80).attr("height", 80)
							o.node = o.node.data(o.nodes).enter().append("image").attr("x", -16).attr("y", (d) -> - 16).attr("width", (d) -> 32).attr("height", 32).attr("class", "node").attr("xlink:href", (d, i) -> o.nodes[i].image.url).on("dblclick", dblclick).on("dragstart", dragstart).call(o.force.drag)						
							o.force.on "tick", tick
						
						window.xinnectGraph = create()
						xinnectGraph.nodes = _nodes
						xinnectGraph.links = _edges
						init()

						### SIGMA JS VARIANTE
						s.graph.addNode node for node in _nodes.filter((n) -> not nodes.filter((m) -> m.id is n.id).length)
						s.graph.addEdge edge for edge in _edges.filter((n) -> not edges.filter((m) -> n.id is m.id).length)

						CustomShapes.init s
						
						sigma.plugins.dragNodes s, s.renderers[0]

						setInterval(() ->
							sigma.plugins.animate(
								s,
								{
									x: prefix + 'x'
									y: prefix + 'y'
									size: prefix + 'size'
									color: prefix + 'color'
								}
							)
						, 2000)
						###

					if force?
						update()

						window.nodes = nodes
						window.edges = edges
			})
			true