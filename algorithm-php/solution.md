# 1. solution

## question

If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.

如果我们列出所有小于10的自然数都是3或5的倍数，我们得到3、5、6和9。这些倍数的和是23。

## part one

```php

    function solution($number)
    {
        $arr = [];
        $three = (int)floor(($number - 1) / 3);
        $five = (int)floor(($number - 1) / 5);

        if ($three >= 1) {
            for ($i = 1; $i <= $three; $i++) {
                $arr[] = $i * 3;
                if ($five >= 1 && $i <= $five) {
                    $arr[] = $i * 5;
                }
            }
        }

        return array_sum(array_unique($arr));
    }

```

## part two

```php

    function solution($number)
    {
        $i = 1;
        $sum = 0;
        $isFive = true;
        while (true) {
            if (($a = 3 * $i) >= $number) {
                break;
            }
            $sum += $a;
            if ($isFive) {
                $b = 5 * $i;
                if (($isFive = $b < $number) && 0 !== $b % 3) {
                    $sum += $b;
                }
            }
            $i++;
        }

        return $sum;
    }

```

## part three

```php

    function solution($number)
    {
        $threes = floor(($number - 1) / 3);
        $fives = floor(($number - 1) / 5);
        $threesAndFives = floor(($number - 1) / 15);

        $callback = function ($static, $times) {
            return $static * ($times + 1) * $times / 2;
        };

        return $callback(3, $threes) + $callback(5, $fives) - $callback(15, $threesAndFives);
    }

```