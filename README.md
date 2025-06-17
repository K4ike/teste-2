# teste-2
nada a declarar
use image::{ImageBuffer, Rgb};
use num_complex::Complex;
use rayon::prelude::*;

const WIDTH: u32 = 1600;
const HEIGHT: u32 = 1200;
const MAX_ITER: u32 = 1000;

/// Calcula se um ponto pertence ao conjunto de Mandelbrot
fn mandelbrot(c: Complex<f64>) -> u32 {
    let mut z = Complex::new(0.0, 0.0);
    for i in 0..MAX_ITER {
        z = z * z + c;
        if z.norm_sqr() > 4.0 {
            return i;
        }
    }
    MAX_ITER
}

/// Gera o fractal e salva como PNG
fn generate_fractal(filename: &str) {
    let mut img = ImageBuffer::new(WIDTH, HEIGHT);

    // Paraleliza o processamento das linhas da imagem
    img.enumerate_rows_mut().par_bridge().for_each(|(y, row)| {
        let y = y as f64;
        for (x, pixel) in row {
            let x = x as f64;
            let cx = (x - WIDTH as f64 / 2.0) * 4.0 / WIDTH as f64;
            let cy = (y - HEIGHT as f64 / 2.0) * 4.0 / HEIGHT as f64;
            let c = Complex::new(cx, cy);

            let iterations = mandelbrot(c);
            let color = if iterations == MAX_ITER {
                Rgb([0, 0, 0]) // Preto para pontos dentro do conjunto
            } else {
                // Mapeia iterações para cores vibrantes
                let r = (iterations % 8) * 32;
                let g = (iterations % 16) * 16;
                let b = (iterations % 32) * 8;
                Rgb([r as u8, g as u8, b as u8])
            };
            *pixel = color;
        }
    });

    img.save(filename).unwrap();
    println!("Fractal salvo como '{}'!", filename);
}

fn main() {
    generate_fractal("mandelbrot.png");
}
