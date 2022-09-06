<script>

// TODO: pass colors from settings to the script
// TODO: hist lines => recangles (like in volbar.js)

import { Overlay } from 'trading-vue-js'

export default {
    name: 'MACDHist',
    mixins: [Overlay],
    methods: {
        meta_info() {
            return {
                author: 'StdSquad', version: '1.0.2',
                desc: 'Moving Average Convergence/Divergence',
                preset: {
                    name: 'MACD $fast $slow $smooth',
                    side: 'offchart',
                    settings: {
                        histWidth: 4,
                        defColor: "#42b28a",
                        histColors: [
                            "#35a776", "#79e0b3", "#e54150", "#ea969e"
                        ]
                    }
                }
            }
        },

        draw(ctx) {
            const layout = this.$props.layout

            // HISTOGRAM

            const base = layout.$2screen(0) + 0.5
            var width = Math.abs(layout.t2screen(this.$props.data[0][0]) - layout.t2screen(this.$props.data[1][0]))
			if (width > 5) {
				width -= 2
			}
			ctx.lineWidth = width

			var oldHist = null;
            for (var p of this.$props.data) {
				let hist = p[1];
				let x = layout.t2screen(p[0])
				let y = layout.$2screen( p[1] )

				var color = ( hist >= 0 ) ? 0 : 2;
				if ( oldHist != null ) {
					if ( hist >= 0 ) {
						color = 0;
						if ( hist < oldHist ) { color = 1; }
					} else {
						color = 2;
						if ( hist > oldHist ) { color = 3; }
					}
				}
				oldHist = hist;
				ctx.strokeStyle = this.sett.histColors[ color ]
				ctx.beginPath()
				ctx.moveTo(x, base)
				ctx.lineTo(x, y)

				ctx.stroke()
            }

			// SEGMENT

			if ( this.segment != null ) {
				this.draw_segment(ctx, this.segment)
			}

			// MARKER

			if ( this.marker != null ) {
				ctx.lineWidth = 1.5
				ctx.strokeStyle = 'black'
				document.body.style.cursor = 'auto'
				this.selected = null
				this.draw_marker(ctx, this.marker)
			}

        },

		draw_segment(ctx, segment) {
			let layout = this.$props.layout

			ctx.lineWidth = segment.lineWidth
			ctx.strokeStyle = segment.color
			ctx.beginPath()

			let x1 = layout.t2screen(segment.p1[0])
			let y1 = layout.$2screen(segment.p1[1])
			if (segment.p1[1] < 0) {
				y1 += segment.lineWidth
			} else {
				y1 -= segment.lineWidth
			}
			ctx.moveTo(x1, y1)

			let x2 = layout.t2screen(segment.p2[0])
			let y2 = layout.$2screen(segment.p2[1])
			if (segment.p2[1] < 0) {
				y2 += segment.lineWidth
			} else {
				y2 -= segment.lineWidth
			}
			ctx.lineTo(x2, y2)

			ctx.stroke()


			ctx.fillStyle = segment.color
			ctx.beginPath();
			if (segment.p1[1] < 0) {
				ctx.moveTo(x1 - segment.lineWidth, y1 + Math.round(segment.lineWidth / 2))
				ctx.lineTo(x1, y1 - Math.round(segment.lineWidth * 1.5));
				ctx.lineTo(x1 + segment.lineWidth, y1 + Math.round(segment.lineWidth / 2));
			} else {
				ctx.moveTo(x1 - segment.lineWidth, y1 - Math.round(segment.lineWidth / 2))
				ctx.lineTo(x1, y1 + Math.round(segment.lineWidth * 1.5));
				ctx.lineTo(x1 + segment.lineWidth, y1 - Math.round(segment.lineWidth / 2));
			}
			ctx.fill();

			ctx.beginPath();
			if (segment.p2[1] < 0) {
				ctx.moveTo(x2 - segment.lineWidth, y2 + Math.round(segment.lineWidth / 2))
				ctx.lineTo(x2, y2 - Math.round(segment.lineWidth * 1.5));
				ctx.lineTo(x2 + segment.lineWidth, y2 + Math.round(segment.lineWidth / 2));
			} else {
				ctx.moveTo(x2 - segment.lineWidth, y2 - Math.round(segment.lineWidth / 2))
				ctx.lineTo(x2, y2 + Math.round(segment.lineWidth * 1.5));
				ctx.lineTo(x2 + segment.lineWidth, y2 - Math.round(segment.lineWidth / 2));
			}
			ctx.fill();
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

        use_for() { return ['MACDHist'] },
        legend(values) {
            let xs = values.slice(1,4).map(x => {
                return x.toFixed(Math.abs(x) > 0.001 ? 4 : 8)
            })
            return [
                {value: xs[0], color: this.hist_colors[values[4]]},
                {value: xs[1], color: this.macd_color},
                {value: xs[2], color: this.signal_color}
            ]
        },
        calc() {
            return {
                props: {
                    fast: { def: 12, text: 'Fast Length' },
                    slow: { def: 26, text: 'Slow Length' },
                    smooth: { def: 9, text: 'Signal EMA' }
                },
                update: `
                    let [macd, signal, hist] =
                        macd(close, fast, slow, smooth)

                    return [hist[0]]
                `
            }
        }
    },
    // Define internal setting & constants here
    computed: {
        sett() {
            return this.$props.settings
        },
        hist_width() {
            return this.sett.histWidth || 4
        },
        hist_colors() {
            return this.sett.histColors
        },
		segment() {
			return this.sett.segment
		},
		marker() {
			return this.sett.marker
		},
		hist_only() {
			return this.sett.histOnly
		}
    }
}
</script>
