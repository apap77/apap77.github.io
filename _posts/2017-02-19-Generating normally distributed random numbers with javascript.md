---
title: "Javascript에서 정규분포를 따르는 난수 발생시키기"
layout: post
comments: true
date: 2017-02-19
---

## Box-Muller transform

Box-Muller transform은 pseudo-random number sampling method의 일종으로, 균일분포를 따르는 난수들(uniformly distributed random numbers)로부터 서로 독립인 한 쌍의 표준 정규 분포를 따르는 난수를 발생시킨다.

$$ U1 $$과 $$ U2 $$는 (0, 1) 사이에 균일하게 분포한 난수이며, 서로 독립이다. 이때 

$$
R^2 = -2 \cdot lnU_1, \theta = 2\pi U_2
$$
라 하면,

$$
Z_0 = Rcos(\theta) = \sqrt{-2lnU_1}cos(2\pi U_2)
$$

그리고

$$
Z_1 = Rcos(\theta) = \sqrt{-2lnU_1}sin(2\pi U_2)
$$

가 된다. 이로부터 얻은 $$ Z_0 $$와 $$ Z_1 $$은 표준정규분포를 따르고, 서로 독립인 난수가 된다.

### Proof)

증명은 생각을 거꾸로 하는 것 부터 시작한다. 

*어떻게 하면 표준정규분포를 따르고, 서로 독립인 난수 $$ Z_0 $$와 $$ Z_1 $$을 좌표 상에 만들어 낼 수 있을까?*


$$ Z_0 \sim N(0, 1) $$

$$ Z_1 \sim N(0, 1) $$

따라서 joint distribution $$ p(z_0, z_1) $$는 다음과 같이 나타난다.

$$
p(z_0, z_1) = p(z_0)p(z_1) = \frac{1}{\sqrt{2\pi}}e^{\frac{-z_0^2}{2}}\frac{1}{\sqrt{2\pi}}e^{\frac{-z_1^2}{2}} = \frac{1}{2\pi}e^{-\frac{z_0^2+z_1^2}{2}}
$$

이때 $$ z_1^2+z_2^2 = r^2 $$ 이므로, 이로부터 joint distribution의 Cartesian representation과 polar representation 사이를 연관지을 수 있다:

$$
p(z_0, z_1) = (\frac{1}{2\pi})(e^{-\frac{r^2}{2}})
$$

우리는 이것을 두 확률밀도함수의 곱으로 생각할 수 있다. 즉, 위의 식을 반지름의 제곱 $$ R^2 $$의 [exponential distribution][1]과($$ (e^{-\frac{r^2}{2}}) $$):

$$
r^2 \sim Exp(\frac{1}{2})
$$

각도 $$ \theta $$의 균일분포($$ \frac{1}{2\pi} $$):

$$
\theta \sim Unif(0, 2\pi)
$$

의 곱으로 생각할 수 있다.

우리가 갖고 있는 것은 균일분포를 따르는 난수들이므로, 현재까지는 균일분포를 따르는 $$ \theta $$를 만들어 낼 수 있다!

이제 마지막으로 해야 할 일은 균일분포를 따르는 난수들로부터 exponential distribution을 따르는 $$ R^2 $$을 만들어 내는 것이다. 이것은 exponential distribution의 정의로부터 쉽게 유도된다(lambda는 exponential distribution의 parameter이다.):

$$
Exp(\lambda) = \frac{-ln(Unif(0, 1))}{\lambda}
$$

따라서

$$
r \sim \sqrt{-2ln(Unif(0, 1))}
$$

이 되어 균일분포를 따르는 난수 두 개로부터 표준정규분포를 따르는 $$ Z_0 $$과 $$ Z_1 $$을 얻어낼 수 있다는 것이 증명되었다.

## Javascript code

Javascript로 정규분포를 따르는 난수를 발생시키기 위해서는 아래의 함수를 사용하면 된다.
{% highlight javascript %}
function random_normal(mean, variance) {
	// normally distributed random number generator
	// using Box-Muller transform
	var z0, r, theta;
	r = Math.sqrt(-2 * Math.log(Math.random()))
	theta = 2 * Math.PI * Math.random()
	z0 = r * Math.cos(theta)

	return mean + variance * z0
}
{% endhighlight %}


## References
[WikiPedia][Wiki]

[https://theclevermachine.wordpress.com/2012/09/11/sampling-from-the-normal-distribution-using-the-box-muller-transform/][2]

[Wiki]: https://en.wikipedia.org/wiki/Box%E2%80%93Muller_transform

[1]: https://en.wikipedia.org/wiki/Exponential_distribution
[2]: https://theclevermachine.wordpress.com/2012/09/11/sampling-from-the-normal-distribution-using-the-box-muller-transform/
