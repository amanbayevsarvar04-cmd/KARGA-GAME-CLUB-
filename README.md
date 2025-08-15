import React, { useEffect, useMemo, useRef, useState } from "react"; import { motion } from "framer-motion"; import { ShoppingCart, Trash2, Percent, Printer, Plus, Minus, Monitor, Clock3, Search, CreditCard, Wallet, Smartphone, Check, X, ReceiptText, Download, History, Settings } from "lucide-react"; import { Button } from "@/components/ui/button"; import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"; import { Input } from "@/components/ui/input"; import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from "@/components/ui/dialog"; import { Label } from "@/components/ui/label"; import { Select, SelectTrigger, SelectContent, SelectItem, SelectValue } from "@/components/ui/select"; import { Badge } from "@/components/ui/badge";

// --- Helper Types --- /** @typedef {{id:string,name:string,price:number,category:string,sku?:string}} Product / /* @typedef {{id:string, name:string, ratePerHour:number, status:"idle"|"running"|"paused", startTime?:number, accumulatedMs?:number}} Station / /* @typedef {{id:string, items:Array<{id:string,name:string,qty:number,price:number,subtotal:number}>, discountPct:number, paymentMethod:string, total:number, paid:number, change:number, at:number}} Order */

// --- Sample Catalog --- const CATALOG: Product[] = [ { id: "p1", name: "Energetik ichimlik", price: 12000, category: "Ichimlik" }, { id: "p2", name: "Kola 0.5L", price: 9000, category: "Ichimlik" }, { id: "p3", name: "Chips", price: 8000, category: "Snack" }, { id: "p4", name: "Hot-dog", price: 15000, category: "Snack" }, ];

// --- Default Stations --- const DEFAULT_STATIONS: Station[] = [ { id: "ST-1", name: "Billiard stoli", ratePerHour: 40000, status: "idle", accumulatedMs: 0 }, { id: "ST-2", name: "PlayStation 1", ratePerHour: 20000, status: "idle", accumulatedMs: 0 }, { id: "ST-3", name: "PlayStation 2", ratePerHour: 20000, status: "idle", accumulatedMs: 0 }, { id: "ST-4", name: "PlayStation 3", ratePerHour: 20000, status: "idle", accumulatedMs: 0 }, { id: "ST-5", name: "VIP xona (PS4)", ratePerHour: 30000, status: "idle", accumulatedMs: 0 }, ];

// --- Utilities --- const fmt = (n: number) => new Intl.NumberFormat("uz-UZ").format(Math.round(n)); const now = () => Date.now(); function msToHM(ms: number) { const totalMinutes = Math.floor(ms / 60000); const h = Math.floor(totalMinutes / 60); const m = totalMinutes % 60; return ${String(h).padStart(2, "0")}:${String(m).padStart(2, "0")}; } function msToText(ms: number) { const totalMinutes = Math.floor(ms / 60000); const h = Math.floor(totalMinutes / 60); const m = totalMinutes % 60; return ${h > 0 ? h + " soat " : ""}${m > 0 ? m + " minut" : ""}.trim(); }

// LocalStorage helpers const LS = { stations: "gc_pos_stations_v1", orders: "gc_pos_orders_v1", };

export default function GameClubPOS() { const [query, setQuery] = useState(""); const [category, setCategory] = useState<string | "barchasi">("barchasi"); const [cart, setCart] = useState<Array<{ id: string; name: string; price: number; qty: number }>>([]); const [discountPct, setDiscountPct] = useState(0); const [paymentMethod, setPaymentMethod] = useState("naqd"); const [paid, setPaid] = useState<number | "">(""); const [orders, setOrders] = useState<Order[]>(() => { try { return JSON.parse(localStorage.getItem(LS.orders) || "[]"); } catch { return []; } }); const [stations, setStations] = useState<Station[]>(() => { try { return JSON.parse(localStorage.getItem(LS.stations) || "null") || DEFAULT_STATIONS; } catch { return DEFAULT_STATIONS; } }); const [settingsOpen, setSettingsOpen] = useState(false); const [receiptOrder, setReceiptOrder] = useState<Order | null>(null);

useEffect(() => { localStorage.setItem(LS.orders, JSON.stringify(orders)); }, [orders]); useEffect(() => { localStorage.setItem(LS.stations, JSON.stringify(stations)); }, [stations]);

const filteredCatalog = useMemo(() => { const q = query.trim().toLowerCase(); return CATALOG.filter((p) => (category === "barchasi" || p.category === category) && (!q || p.name.toLowerCase().includes(q)) ); }, [query, category]);

const subtotal = cart.reduce((s, it) => s + it.price * it.qty, 0); const discountValue = Math.min(Math.max(discountPct, 0), 100) * subtotal / 100; const total = Math.max(subtotal - discountValue, 0); const change = typeof paid === "number" ? Math.max(paid - total, 0) : 0;

const addToCart = (p: { id: string; name: string; price: number }, qty = 1) => { setCart((c) => { const idx = c.findIndex((x) => x.id === p.id); if (idx >= 0) { const next = [...c]; next[idx] = { ...next[idx], qty: next[idx].qty + qty }; return next; } return [...c, { ...p, qty }]; }); };

const clearCart = () => { setCart([]); setDiscountPct(0); setPaid(""); };

const placeOrder = () => { if (cart.length === 0) return; const items = cart.map((it) => ({ id: it.id, name: it.name, qty: it.qty, price: it.price, subtotal: it.qty * it.price })); const order: Order = { id: ORD-${Date.now()}, items, discountPct, paymentMethod, total, paid: typeof paid === "number" ? paid : 0, change: typeof paid === "number" ? Math.max(paid - total, 0) : 0, at: Date.now(), }; setOrders((o) => [order, ...o]); setReceiptOrder(order); clearCart(); };

const startStation = (id: string) => setStations((arr) => arr.map((s) => s.id === id ? ({ ...s, status: "running", startTime: now(), }) : s));

const stopStationToCart = (id: string) => setStations((arr) => arr.map((s) => { if (s.id !== id) return s; const usedMs = (s.accumulatedMs || 0) + (s.startTime ? now() - s.startTime : 0); const hours = usedMs / 3_600_000; const price = Math.ceil(hours * s.ratePerHour); addToCart({ id: TIME-${s.id}-${Date.now()}, name: ${s.name} (${msToText(usedMs)}), price }, 1); return { ...s, status: "idle", startTime: undefined, accumulatedMs: 0 }; }));

return <div>Karga Game Club POS</div>; }

