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
