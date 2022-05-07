<script>
	const timeratio = 2.5;
	const igMinuteOffset = 5;
	var hour = `00`,
		time = `00`;

	function getIgTime() {
		let server_time = new Date();

		let serverMinutes = server_time.getUTCMinutes(),
			serverSeconds = server_time.getUTCSeconds(),
			totalSeconds = serverMinutes * 60 + serverSeconds;

		let ratio = totalSeconds / timeratio;

		let _igHour = Math.trunc(ratio / 60),
			_igMinute = Math.trunc(ratio - _igHour * 60);

		_igHour = _igMinute == 0 && _igHour - 1 >= 0 ? _igHour - 1 : _igHour;
		_igMinute = _igMinute == 0 ? 60 - igMinuteOffset : _igMinute < igMinuteOffset ? 60 + _igMinute - igMinuteOffset : _igMinute - igMinuteOffset;

		hour = formatTime(_igHour);
		time = formatTime(_igMinute);
	}

	const formatTime = (x) => x.toString().padStart(2, `0`);

	getIgTime();
	setInterval(getIgTime, 500);
</script>

<div class="clock">
	<div id="hour">{hour}</div>
	<div>:</div>
	<div id="minute">{time}</div>
</div>

<style>
	.clock {
		display: flex;
		color: var(--orange-light);
	}
</style>
