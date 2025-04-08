const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const ImageMinimizerPlugin = require('image-minimizer-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const WebpackObfuscator = require('webpack-obfuscator');
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
    mode: 'production',
    entry: {
        main: './assets/webpack/index.js',
        app: './assets/js/app.js',
    },

    output: {
        filename: '[name].[contenthash].js',
        path: path.resolve(__dirname, 'assets/dist'),
        assetModuleFilename: 'images/[name][contenthash][ext]',
    },
    devServer: {
        static: path.resolve(__dirname, 'assets/dist'),
        hot: true,
    },
    module: {
        rules: [
            {
                test: /\.scss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    'sass-loader',
                ],
            },
            {
                test: /\.(png|jpe?g|gif|svg)$/i,
                type: 'asset/resource',
                include: path.resolve(__dirname, 'assets/images'),
            }
        ],
    },
    optimization: {
        minimize: true,
        minimizer: [
            `...`,
            new CssMinimizerPlugin(),
            new TerserPlugin({
                terserOptions: {
                    format: {
                        comments: false,
                    },
                    compress: {
                        drop_console: true,
                        drop_debugger: true,
                        passes: 5,
                        booleans_as_integers: true,
                        unsafe: true,
                        unsafe_math: true,
                        unsafe_proto: true,
                    },
                    mangle: {
                        toplevel: true,
                        properties: {
                            regex: /.*/,
                        },
                    },
                },
                extractComments: false,
            }),
            new WebpackObfuscator(
                {
                    rotateStringArray: true,
                    stringArray: true,
                    stringArrayThreshold: 0.75,
                    debugProtection: true,
                    debugProtectionInterval: 1000,
                    disableConsoleOutput: true,
                },
                ['main.js']
            ),
            //new CleanWebpackPlugin(),
        ],
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'style.css',
        }),
        new CopyWebpackPlugin({
            patterns: [
                {
                    from: path.resolve(__dirname, 'assets/images'),
                    to: path.resolve(__dirname, 'assets/dist/images'),
                },
            ],
        }),
        new ImageMinimizerPlugin({
            minimizer: {
                implementation: ImageMinimizerPlugin.sharpMinify,
                options: {
                    encodeOptions: {
                        jpeg: { quality: 75 },
                        png: { quality: 80 },
                    },
                },
            },
        }),
    ],
    watch: true,
};
