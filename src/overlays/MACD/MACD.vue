<script>

// TODO: pass colors from settings to the script
// TODO: hist lines => recangles (like in volbar.js)

import { Overlay } from 'trading-vue-js'

export default {
    name: 'MACD',
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
                        macdWidth: 1,
                        signalWidth: 1,
                        defColor: "#42b28a",
                        macdColor: "#3782f2",
                        signalColor: "#f48709",
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

			// MACD LINE

            ctx.beginPath()

            ctx.lineWidth = this.macd_width
            ctx.strokeStyle = this.macd_color

            for (var p of this.$props.data) {
                let x = layout.t2screen(p[0])
                let y = layout.$2screen(p[2])
                ctx.lineTo(x, y)
            }

            ctx.stroke()

            // SIGNAL LINE

            ctx.beginPath()

            ctx.lineWidth = this.signal_width
            ctx.strokeStyle = this.signal_color

            for (var p of this.$props.data) {
                let x = layout.t2screen(p[0])
                let y = layout.$2screen(p[3])
                ctx.lineTo(x, y)
            }

            ctx.stroke()

			// SEGMENT

			if ( this.segment != null ) {
				ctx.lineWidth = this.segment.lineWidth
				ctx.strokeStyle = this.segment.color
				ctx.beginPath()

				let x1 = layout.t2screen(this.segment.p1[0])
				let y1 = layout.$2screen(this.segment.p1[1])
				if (this.segment.p1[1] < 0) {
					y1 += this.segment.lineWidth
				} else {
					y1 -= this.segment.lineWidth
				}
				ctx.moveTo(x1, y1)

				let x2 = layout.t2screen(this.segment.p2[0])
				let y2 = layout.$2screen(this.segment.p2[1])
				if (this.segment.p2[1] < 0) {
					y2 += this.segment.lineWidth
				} else {
					y2 -= this.segment.lineWidth
				}
				ctx.lineTo(x2, y2)

				ctx.stroke()


				ctx.fillStyle = this.segment.color
				ctx.beginPath();
				if (this.segment.p1[1] < 0) {
					ctx.moveTo(x1 - this.segment.lineWidth, y1 + Math.round(this.segment.lineWidth / 2))
					ctx.lineTo(x1, y1 - Math.round(this.segment.lineWidth * 1.5));
					ctx.lineTo(x1 + this.segment.lineWidth, y1 + Math.round(this.segment.lineWidth / 2));
				} else {
					ctx.moveTo(x1 - this.segment.lineWidth, y1 - Math.round(this.segment.lineWidth / 2))
					ctx.lineTo(x1, y1 + Math.round(this.segment.lineWidth * 1.5));
					ctx.lineTo(x1 + this.segment.lineWidth, y1 - Math.round(this.segment.lineWidth / 2));
				}
				ctx.fill();

				ctx.beginPath();
				if (this.segment.p2[1] < 0) {
					ctx.moveTo(x2 - this.segment.lineWidth, y2 + Math.round(this.segment.lineWidth / 2))
					ctx.lineTo(x2, y2 - Math.round(this.segment.lineWidth * 1.5));
					ctx.lineTo(x2 + this.segment.lineWidth, y2 + Math.round(this.segment.lineWidth / 2));
				} else {
					ctx.moveTo(x2 - this.segment.lineWidth, y2 - Math.round(this.segment.lineWidth / 2))
					ctx.lineTo(x2, y2 + Math.round(this.segment.lineWidth * 1.5));
					ctx.lineTo(x2 + this.segment.lineWidth, y2 - Math.round(this.segment.lineWidth / 2));
				}
				ctx.fill();

			}

        },
        use_for() { return ['MACD'] },
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

                    return [hist[0], macd[0], signal[0]]
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
        macd_width() {
            return this.sett.macdWidth || 1
        },
        signal_width() {
            return this.sett.signalWidth || 1
        },
        color() {
            return this.sett.defColor || "#42b28a"
        },
        macd_color() {
            return this.sett.macdColor || "#3782f2"
        },
        signal_color() {
            return this.sett.signalColor || "#f48709"
        },
        hist_colors() {
            return this.sett.histColors
        },
		segment() {
			return this.sett.segment
		},
		hist_only() {
			return this.sett.histOnly
		}
    }
}
</script>
