Login Template Handling and Eventing

	if Meteor.isClient
		Template.graph.rendered = () ->
			el = $(".graph")
			w = el.width()
			h = el.height()

			window.force = d3.layout.force()
				.size([w, h])
				.charge(-400)
				.linkDistance(40)
			###
			window.s = new sigma 
				renderer:
					container: document.getElementById "graph"
					type: "canvas"
				settings:
					edgeColor: 'default'
					defaultEdgeColor: 'grey'
					defaultNodeColor: 'teal'
					animationsTime: 1000
					minNodeSize: 16
					maxNodeSize: 32

			sigma.utils.pkg "sigma.canvas.nodes"
			###

		Template.loadingSwitch.events {}
		Template.layout.helpers {
			dummy: () -> 
				if location.pathname.indexOf("cockpit") is -1 then "dummy"
				else "xing"
		}
		Template.anonymousSwitch.events {
			"click .login.button": (ev, $) -> 
				if Meteor.user()?.profile?.contacts?.length and Meteor.user()?.profile?.avatar? then Router.go "cockpit"
				else Meteor.loginWithXing () -> Router.go "cockpit"
		}
		Template.personalSwitch.events {
			"click .avatar:not(.active):not(.shadow)": () -> Meteor.logout () -> Router.go "security"
			"click .multitasking.icon.link": () -> 
				state = Session.get "trashy"
				Session.set "trashy", !state
				Meteor.call "volatileGraph", !state
		}
		Template.personalSwitch.helpers {
			avatar: () -> Meteor.user().profile.avatar
			loading: () -> Session.get "loading"
			trashy: () -> Session.get "trashy"
		}
		Template.contacts.events {
			"click .facebook.link": () -> window.open "http://www.facebook.com/johannes.neugschwentner", "_blank"
			"click .google.plus.link": () -> window.open "http://google.com/+JohannesNeugschwentner", "_blank"
			"click .mail.link": () -> location.href = "mailto:johannes.neugschwentner@gmail.com"
			"mouseover .circular.avatar.segment": (ev, o) ->  
				$(ev.target).draggable { appendTo: "body", zIndex: "9999999", helper: "clone" }
				$(".vertical.divider > .circular.avatar.segment").droppable
					activeClass: "active"
					hoverClass: "shadow"
					tolerance: "touch"
					drop: (ev, o) -> 
						Session.set "loading", true
						Meteor.call "addConnection", o.draggable.data("id"), (err, result) -> 
							Session.set "loading", false
		}
		Template.dummys.helpers {
			users: () -> Session.get "users"
		}
		Template.contacts.helpers {
			contacts: () -> Session.get "contacts"
		}
		Template.comments.helpers {
			comments: () -> Session.get "comments"
			journalists: () -> Session.get "journalists"
		}

		Template.comments.events {
			"click div.item.move.up.reveal": (ev, $) -> 
				$.$($.firstNode)?.find(".ui.item.move.up.reveal")?.find(".active")?.removeClass "active"
				base = $.$(ev.target).parent ".ui.item.move.up.reveal"
				base.find("div.image.visible.content").addClass "active"
				base.find("i.empty.checkbox.icon").addClass "active"
			"click .submit.button": (ev, $) ->
				base = $.$($.firstNode)
				active = base.find(".ui.item.move.up.reveal .active")
				if active.size() > 0
					name = active.siblings("div.name").text()
					msg = base.find("textarea").val()
					if msg.length > 0
						base.find("textarea").val("")
						Comments.insert {
							name: name
							avatar: name.toLowerCase() + ".jpg"
							comment: msg
						}

		}
