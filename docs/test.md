<!-- 在Markdown文件中嵌入HTML和JavaScript -->

## Interactive Number Doubler

<div class="container">
    <button id="decrement">-</button>
    <input type="number" id="numberInput" value="1" min="1" max="10">
    <button id="increment">+</button>
    <p>Double: <span id="doubleValue">2</span></p>
</div>

<script>
    document.addEventListener("DOMContentLoaded", function() {
        const numberInput = document.getElementById("numberInput");
        const doubleValue = document.getElementById("doubleValue");

        document.getElementById("increment").addEventListener("click", function() {
            if (numberInput.value < 10) {
                numberInput.value++;
                updateDoubleValue();
            }
        });

        document.getElementById("decrement").addEventListener("click", function() {
            if (numberInput.value > 1) {
                numberInput.value--;
                updateDoubleValue();
            }
        });

        numberInput.addEventListener("input", function() {
            if (numberInput.value < 1) numberInput.value = 1;
            if (numberInput.value > 10) numberInput.value = 10;
            updateDoubleValue();
        });

        function updateDoubleValue() {
            doubleValue.textContent = numberInput.value * 2;
        }
    });
</script>
