# Embedded Graphics on Arbitrum Stylus

This repo contains an example on deploying a Rust embedded graphics library as a smart contract to the [Arbitrum Stylus](https://medium.com/offchainlabs/stylus-now-live-one-chain-many-languages-eee56ad7266d) testnet just for fun! 

Although it does nothing with the input at the moment, it is available at address `0x7236441bf4e4c211e4bcc62bd86e1922e351129d` and accessible via the testnet JSON-RPC URL: https://stylus-testnet.arbitrum.io/rpc. Here were the transaction to deploy it and activate it onchain:


```
deploy: 0x89b54d216793c3e0ff5dd1c40330d4abea1690d5517dcc65ae8d4a1d5dd1d293
activate: 0xaccfcc06c5acd653863843389eb83399193ffccf5eafd80913a64509a304dfb5
```

Imagine doing _this_ in Solidity:

```rust
use embedded_graphics::{
    mock_display::MockDisplay,
    mono_font::{ascii::FONT_6X10, MonoTextStyle},
    pixelcolor::BinaryColor,
    prelude::*,
    primitives::{
        Circle, PrimitiveStyle, PrimitiveStyleBuilder, Rectangle, StrokeAlignment, Triangle,
    },
    text::{Alignment, Text},
};

#[entrypoint]
fn user_main(input: Vec<u8>) -> Result<Vec<u8>, Vec<u8>> {
    // Create a new mock display
    let mut display: MockDisplay<BinaryColor> = MockDisplay::new();

    // Create styles used by the drawing operations.
    let thin_stroke = PrimitiveStyle::with_stroke(BinaryColor::On, 1);
    let thick_stroke = PrimitiveStyle::with_stroke(BinaryColor::On, 3);
    let border_stroke = PrimitiveStyleBuilder::new()
        .stroke_color(BinaryColor::On)
        .stroke_width(3)
        .stroke_alignment(StrokeAlignment::Inside)
        .build();
    let fill = PrimitiveStyle::with_fill(BinaryColor::On);
    let character_style = MonoTextStyle::new(&FONT_6X10, BinaryColor::On);

    let yoffset = 10;

    // Draw a 3px wide outline around the display.
    display
        .bounding_box()
        .into_styled(border_stroke)
        .draw(&mut display)
        .unwrap();

    // Draw a triangle.
    Triangle::new(
        Point::new(16, 16 + yoffset),
        Point::new(16 + 16, 16 + yoffset),
        Point::new(16 + 8, yoffset),
    )
    .into_styled(thin_stroke)
    .draw(&mut display)
    .unwrap();

    // Draw a filled square
    Rectangle::new(Point::new(52, yoffset), Size::new(16, 16))
        .into_styled(fill)
        .draw(&mut display)
        .unwrap();

    // Draw a circle with a 3px wide stroke.
    Circle::new(Point::new(88, yoffset), 17)
        .into_styled(thick_stroke)
        .draw(&mut display)
        .unwrap();

    // Draw centered text.
    let text = "embedded-graphics";
    Text::with_alignment(
        text,
        display.bounding_box().center() + Point::new(0, 15),
        character_style,
        Alignment::Center,
    )
    .draw(&mut display)
    .unwrap();

    Ok(input)
}

```