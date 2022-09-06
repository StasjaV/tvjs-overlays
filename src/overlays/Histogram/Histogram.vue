<script>

import { Overlay } from 'trading-vue-js'

export default {
    name: 'Histogram',
    mixins: [Overlay],
    methods: {
        meta_info() {
            return {
                author: 'StdSquad', version: '1.0.0',
                desc: 'Histogram plot'
            }
        },

        draw(ctx) {
			const layout = this.$props.layout
            const base = layout.$2screen(0) + 0.5
            ctx.strokeStyle = this.color

			var width = Math.abs(layout.t2screen(this.$props.data[0][0]) - layout.t2screen(this.$props.data[1][0]))
			if (width > 5) {
				width -= 2
			}

			ctx.lineWidth = width

            ctx.beginPath()

            // Color changed
            let changed = false
            for (var p of this.$props.data) {

				let x = layout.t2screen(p[0])
                let y =  layout.$2screen(p[1])

                let changed = false

                if (p[2]) {
                    if (ctx.strokeStyle !== p[2]) {
                        ctx.stroke()
                        changed = true
                    }
                    ctx.strokeStyle = p[2]
                } else {
                    if (ctx.strokeStyle !== this.color) {
                        ctx.stroke()
                        changed = true
                    }
                    ctx.strokeStyle = this.color
                }
                if (changed) {
					ctx.beginPath()
				}

				ctx.moveTo(x, base)
                ctx.lineTo(x, y)
            }
            ctx.stroke()

			if ( this.marker != null ) {
	            this.draw_marker(ctx, this.marker)
			}
        },

		draw_marker(ctx, p) {
			if (p[1].size == null)
			{
				if (p[1].sel) {
					p[1].size = { height: 20, width: 17, font: 15 };
				} else {
					p[1].size = { height: 14, width: 13, font: 11 };
				}
			}

			if (p[1].position == null) {
				p[1].position = [0, 0]
			}

			if (p[1].direction == null) {
				p[1].direction = "down"
			}

            let layout = this.$props.layout
            let stroke = this.colors.back

            let fill = p[1].color || 'orange'
            let radius = 2
            let height = p[1].size.height
            let width = p[1].size.width
            let x = layout.t2screen(p[0]) - width * 0.5 + p[1].position[0]

            let y = layout.$2screen(p[1].$) + p[1].position[1]
			if (p[1].direction == "down") {
				y = y - (height + height / 5)
			} else {
				y = y + (height + height / 5)
			}

            // Collisions
			if (p[1].direction == "down" &&
					this.mouse.x > x && this.mouse.x < x + width &&
            		this.mouse.y > y && this.mouse.y < y + height
				|| p[1].direction == "up" &&
					this.mouse.x > x && this.mouse.x < x + width &&
	                this.mouse.y < y && this.mouse.y > y - height
			) {
                document.body.style.cursor = 'pointer'
                this.selected = p
                stroke = this.colors.text
            }

            ctx.beginPath()

			if (p[1].direction == "down") {
	            ctx.moveTo(x + radius, y)
	            ctx.lineTo(x + width - radius, y)
	            ctx.quadraticCurveTo(x + width, y, x + width, y + radius)
	            ctx.lineTo(x + width, y + height - radius)
	            ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height)
	            ctx.lineTo(x + width * 1 / 2, y + height * 1.2)
	            ctx.lineTo(x + radius, y + height);
	            ctx.quadraticCurveTo(x, y + height, x, y + height - radius)
	            ctx.lineTo(x, y + radius)
	            ctx.quadraticCurveTo(x, y, x + radius, y)
			} else {
				ctx.moveTo(x + radius, y)
	            ctx.lineTo(x + width - radius, y)
	            ctx.quadraticCurveTo(x + width, y, x + width, y - radius)
	            ctx.lineTo(x + width, y - height + radius)
	            ctx.quadraticCurveTo(x + width, y - height, x + width - radius, y - height)
	            ctx.lineTo(x + width * 1 / 2, y - height * 1.2)
	            ctx.lineTo(x + radius, y - height);
	            ctx.quadraticCurveTo(x, y - height, x, y - height + radius)
	            ctx.lineTo(x, y - radius)
	            ctx.quadraticCurveTo(x, y, x + radius, y)
			}

            ctx.lineWidth = 1
            ctx.closePath()
            ctx.fillStyle = fill
            ctx.strokeStyle = stroke
            ctx.fill()
            ctx.stroke()

            ctx.textAlign ='center'
            ctx.fillStyle = p[1].textColor || this.colors.back
            ctx.font = `${ p[1].size.font }px Arial`
            ctx.fillText(p[1].text || '$', x + width/2, (p[1].direction == "down" ? y + height * 0.8 : y - height * 0.3))

        },

        use_for() { return ['Histogram'] },

        legend(values) {
            let x = values[1]
            if (typeof x === "number") {
                x = x.toFixed(Math.abs(x) > 0.001 ? 4 : 8)
            }
            return [{
                value: x,
                color:values[2] || this.color
            }]
        },
    },
    // Define internal setting & constants here
    computed: {
        sett() {
            return this.$props.settings
        },
        line_width() {
            return this.sett.lineWidth || 2
        },
        color() {
            const n = this.$props.num % 5
            return this.sett.color || this.COLORS[n]
        },
		marker() {
			return this.sett.marker
		}
    },
    data() {
        return {
            COLORS:
            [
                '#888888', '#42b28a', '#5691ce', '#612ff9',
                '#d50b90', '#ff2316'
            ]
        }
    }

}
</script>
