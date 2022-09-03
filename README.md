# constfft
A Rust library crate providing an [FFT] API for arrays. This crate wraps the
[rustfft] and [realfft] crates that does the heavy lifting behind the scenes.
Use `constfft` if you're working with [arrays], i.e. you know the size of the
signal at compile time.

### Complex Signal Example:
```rust
use approx::assert_ulps_eq;
use constfft::Complex;
use constfft::Fft;
use constfft::Ifft;

fn main() {
    // Define a real-valued signal
    let real_signal = [1.0_f64; 10];

    // Call `.fft` on the signal to obtain it's discrete fourier transform
    let real_signal_dft: [Complex<f64>; 10] = real_signal.fft();

    // Call `.ifft` on the frequency domain signal to obtain it's inverse
    let real_signal_dft_idft: [Complex<f64>; 10] = real_signal_dft.ifft();

    // Verify the resulting signal is a scaled value of the original signal
    for (original, manipulated) in real_signal.iter().zip(real_signal_dft_idft) {
        assert_ulps_eq!(manipulated.re, original * real_signal.len() as f64);
        assert_ulps_eq!(manipulated.im, 0.0);
    }

    // Define a complex-valued signal
    let complex_signal = [Complex::new(1.0_f64, 0.0); 10];

    // Call `.fft` on the signal to obtain it's discrete fourier transform
    let complex_signal_dft: [Complex<f64>; 10] = complex_signal.fft();

    // Call `.ifft` on the frequency domain signal to obtain it's inverse
    let complex_signal_dft_idft: [Complex<f64>; 10] = complex_signal_dft.ifft();

    // Verify the resulting signal is a scaled value of the original signal
    for (original, manipulated) in complex_signal.iter().zip(complex_signal_dft_idft) {
        assert_ulps_eq!(manipulated.re, original.re * complex_signal.len() as f64);
        assert_ulps_eq!(manipulated.im, original.im * complex_signal.len() as f64);
    }
}
```

### Real Signal Example:
```rust
#![allow(incomplete_features)]
#![feature(generic_const_exprs)]

use approx::assert_ulps_eq;
use constfft::RealFft;
use constfft::RealIfft;

fn main() {
    // Define a real-valued signal
    let real_signal = [1.0_f64; 10];
    // Call `.real_fft()` on the signal to obtain it's discrete fourier transform
    let real_signal_dft = real_signal.real_fft();
    // Call `.real_ifft` on the RealDft signal to obtain it's real inverse
    let real_signal_dft_idft: [f64; 10] = real_signal_dft.real_ifft();

    // Verify the resulting ifft is a scaled version of the original signal
    for (original, manipulated) in real_signal.iter().zip(real_signal_dft_idft) {
        assert_ulps_eq!(manipulated, original * 10.0);
    }
}
```

### Advantages
* If it compiles, your code **won't panic™** in this library[^panic]
* Ergonomic API

### Current limitations
* Requires nightly because it uses the [generic_const_exprs] feature
* No implementation for slices

### Possible future plans
Currently I don't see any reason why the same API (minus the compile time size
checks) cannot be implementable on slices. This seems like a much nicer API to
work with and AFAICT does not depend on the types to be arrays. If this is the
case I'd implement the same API on slices and rename this crate to `easyfft`.

#### Footnotes
[^panic]: While this could be true in theory, in practice it most probably is not.
Currently you can create a `RealDft` struct and mutate it using the `DerefMut`
trait in a way that the `real_ifft` method will cause a panic. This should be
fixable in a future patch release currently pending implementation details.
There could be other bugs in this crate or it's dependencies that may cause a
panic, but in theory all the runtime panics have been moved to compile time
errors.

[FFT]: https://en.wikipedia.org/wiki/Fast_Fourier_transform
[rustfft]: https://docs.rs/rustfft/latest/rustfft/
[realfft]: https://docs.rs/realfft/latest/realfft/
[arrays]: https://doc.rust-lang.org/std/primitive.array.html
[generic_const_exprs]: https://github.com/rust-lang/rust/issues/76560
